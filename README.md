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
