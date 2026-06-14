 from rest_framework import status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import AllowAny
from rest_framework.response import Response
from django.core.exceptions import ValidationError
from django.http import JsonResponse
from .models import User, Post


@api_view(['POST'])
@permission_classes([AllowAny])
def login(request):
    try:
        email = request.data.get('email')
        password = request.data.get('password')

        if not email or not password:
            return Response(
                {'message': 'Email and password are required'}, 
                status=status.HTTP_400_BAD_REQUEST
            )

        user = User.objects.findByEmail(email)

        if not user:
            return Response(
                {'message': 'Invalid credentials'}, 
                status=status.HTTP_401_UNAUTHORIZED
            )

        if user.password != password:
            return Response(
                {'message': 'Invalid credentials'}, 
                status=status.HTTP_401_UNAUTHORIZED
            )

        user_profile = user.toPublicProfile()

        return Response({
            'message': 'Login successful',
            'user': user_profile
        }, status=status.HTTP_200_OK)

    except Exception as e:
        print(f'Login error: {e}')
        return Response(
            {'message': 'Server error during login'}, 
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )


@api_view(['GET'])
def get_current_user(request):
    try:
        user_id = request.headers.get('x-user-id')

        if not user_id:
            return Response(
                {'message': 'Not authenticated'}, 
                status=status.HTTP_401_UNAUTHORIZED
            )

        try:
            user = User.objects.get(id=user_id)
        except User.DoesNotExist:
            return Response(
                {'message': 'User not found'}, 
                status=status.HTTP_404_NOT_FOUND
            )

        return Response({'user': user.toPublicProfile()}, status=status.HTTP_200_OK)

    except Exception as e:
        print(f'Get current user error: {e}')
        return Response(
            {'message': 'Server error'}, 
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )


@api_view(['GET', 'POST'])
@permission_classes([AllowAny])
def posts_view(request):
    if request.method == 'GET':
        return get_all_posts(request)
    elif request.method == 'POST':
        return create_post(request)


def get_all_posts(request):
    try:
        posts = Post.findAll()
        posts_data = [post.to_dict() for post in posts]

        return Response({'posts': posts_data}, status=status.HTTP_200_OK)

    except Exception as e:
        print(f'Get all posts error: {e}')
        return Response(
            {'message': 'Server error fetching posts'},
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )


@api_view(['GET', 'PUT', 'DELETE'])
@permission_classes([AllowAny])
def post_detail_view(request, id):
    if request.method == 'GET':
        return get_post_by_id(request, id)
    elif request.method == 'PUT':
        return update_post(request, id)
    elif request.method == 'DELETE':
        return delete_post(request, id)


def get_post_by_id(request, id):
    try:
        try:
            int(id)
        except ValueError:
            return Response(
                {'message': 'Invalid post ID'}, 
                status=status.HTTP_400_BAD_REQUEST
            )

        try:
            post = Post.objects.get(id=id)
        except Post.DoesNotExist:
            return Response(
                {'message': 'Post not found'}, 
                status=status.HTTP_404_NOT_FOUND
            )

        post_data = post.to_dict()
        return Response({'post': post_data}, status=status.HTTP_200_OK)

    except Exception as e:
        print(f'Get post by ID error: {e}')
        return Response(
            {'message': 'Server error fetching post'}, 
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )


@api_view(['GET'])
def get_my_posts(request):
    try:
        user_id = request.headers.get('x-user-id')

        if not user_id:
            return Response(
                {'message': 'Not authenticated'}, 
                status=status.HTTP_401_UNAUTHORIZED
            )

        posts = Post.findByAuthor(user_id)
        posts_data = [post.to_dict() for post in posts]

        return Response({'posts': posts_data}, status=status.HTTP_200_OK)

    except Exception as e:
        print(f'Get my posts error: {e}')
        return Response(
            {'message': 'Server error fetching posts'}, 
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )


def create_post(request):
    try:
        user_id = request.headers.get('x-user-id')
        if not user_id:
            return Response(
                {'message': 'Not authenticated'}, 
                status=status.HTTP_401_UNAUTHORIZED
            )

        if not title or not content or not category:
            return Response(
                {'message': 'Title, content, and category are required'}, 
                status=status.HTTP_400_BAD_REQUEST
            )

        if not excerpt:
            excerpt = content[:150] if len(content) > 150 else content

        if tags is None:
            tags = []

        if not readTime:
            readTime = max(1, len(content.split()) // 200)  # ~200 words per minute

        post = Post.objects.create(
            title=title,
            content=content,
            excerpt=excerpt,
            category=category,
            tags=tags,
            readTime=readTime,
            author_id=user_id
        )


        return Response({
            'message': 'Post created successfully',
            'post': post
        }, status=status.HTTP_201_CREATED)

    except Exception as e:
        print(f'Create post error: {e}')
        return Response(
            {'message': 'Server error creating post'}, 
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )


def update_post(request, id):
    try:
        user_id = request.headers.get('x-user-id')

        if not user_id:
            return Response(
                {'message': 'Not authenticated'}, 
                status=status.HTTP_401_UNAUTHORIZED
            )

        try:
            int(id)
        except ValueError:
            return Response(
                {'message': 'Invalid post ID'}, 
                status=status.HTTP_400_BAD_REQUEST
            )

        try:
            post = Post.objects.get(id=id)
        except Post.DoesNotExist:
            return Response(
                {'message': 'Post not found'}, 
                status=status.HTTP_404_NOT_FOUND
            )

        if str(post.author.id) != user_id:
            return Response(
                {'message': 'Not authorized to update this post'}, 
                status=status.HTTP_403_FORBIDDEN
            )

        title = request.data.get('title')
        content = request.data.get('content')
        excerpt = request.data.get('excerpt')
        category = request.data.get('category')
        tags = request.data.get('tags')
        readTime = request.data.get('readTime')
        published = request.data.get('published')

        if title:
            post.title = title
        if content:
            post.content = content
            if not readTime:
                word_count = len(content.split())
                post.readTime = max(1, word_count // 200)
        if excerpt is not None:
            post.excerpt = excerpt
        if category:
            post.category = category
        if tags is not None:
            post.tags = tags
        if readTime is not None:
            post.readTime = readTime
        if published is not None:
            post.published = published

        post.save()

        post_data = post.to_dict()

        return Response({
            'message': 'Post updated successfully',
            'post': post_data
        }, status=status.HTTP_200_OK)

    except Exception as e:
        print(f'Update post error: {e}')
        return Response(
            {'message': 'Server error updating post'}, 
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )


def delete_post(request, id):
    try:
        user_id = request.headers.get('x-user-id')

        if not user_id:
            return Response(
                {'message': 'Not authenticated'}, 
                status=status.HTTP_401_UNAUTHORIZED
            )

        try:
            int(id)
        except ValueError:
            return Response(
                {'message': 'Invalid post ID'}, 
                status=status.HTTP_400_BAD_REQUEST
            )

        try:
            post = Post.objects.get(id=id)
        except Post.DoesNotExist:
            return Response(
                {'message': 'Post not found'}, 
                status=status.HTTP_404_NOT_FOUND
            )

        if str(post.author.id) != user_id:
            return Response(
                {'message': 'Not authorized to delete this post'}, 
                status=status.HTTP_403_FORBIDDEN
            )

        post.delete()

        return Response({
            'message': 'Post deleted successfully'
        }, status=status.HTTP_200_OK)

    except Exception as e:
        print(f'Delete post error: {e}')
        return Response(
            {'message': 'Server error deleting post'}, 
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )
