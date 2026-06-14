import jwt

from django.conf import settings

from django.http import JsonResponse

from functools import wraps

from .models import User
JWT_SECRET = getattr(settings, 'JWT_SECRET', "workfLow-secret-key-change-in-pi
def generate_token(user_id) :
import datetime
now = datetime.datetime.now(datetime.UTC)
payload = {
"userId': str(user_id),
"exp*: now + datetime.timedelta(days=7),
"iat': now
return jwt.encode(payLoad, JWT_SECRET, algorithm= "HS256°)

def authenticate(view_func):
@wraps (view_func)
def wrapper (request, *args, **kwargs):
try:
auth_header = request. headers.get("Authorization', ")
if not auth_header or not auth_header.startswith("Bearer '): return JsonResponse(f"message': "Authentication required"}, stat
token = auth_header.split( ")[1]
if not token:
return JsonResponse({message': "Authentication required"}, status
56
37
38
39
46
41
try:
decoded = jwt. decode(token, JWT_SECRET, algorithms=[HS256'])
except jwt.ExpiredSignatureError:
return JsonResponse(fmessage": "Token expired"}, status=401)
except jwt.InvaLidTokenError:
Problems
Output
return JsonResponse(f"message': "Invalid token'}, status=401)

except jwt.ExpiredSignatureError:
40
return JsonResponse(f"message': "Token expired"}, status=401)
except jwt. InvalidTokenError:
41
42
return JsonResponse(f"message': "Invalid token'}, status=401)
43
44
user_id = decoded.get('userId')
45
try:
46
47
48
49
user = User. objects.get(id=int(user_id))
except (User.DoesNotExist, ValueError):
return JsonResponse(fmessage': "User not found'}, status=401)
50
51
request.user = user
52
53
return view_func(request, *args,
**kwargs)
54
55
56
57
except Exception as g:
return JsonResponse("message*: "Authentication error'}, status=500)
return wrapper
58
5오
Problems
Output Debug Console
Terminal
Ports



68
61
62
63
64
65
66
67
return wrapper
def is_admin(view_func):
@wraps (view_func)
def wrapper (request, *args,
**kwargs):
if request.user role ==
'admin':
return JsonResponse ({'message':
return view_func(request, *args,
•Admin
**kwargs)
return wrapper
