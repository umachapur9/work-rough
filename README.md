import json
from django.http import JsonResponse from from
django.views.decorators.http import require_http_methods django.views.decorators.csrf import csrf_exempt
from json import JSONDecodeError

from .•models import Team from ..middleware import authenticate
@csrf_exempt
@require_http_methods(["GET"])
authenticate
def get_all_teams (request):
try:
teams = Team,objects,prefetch_related("members").all()
teams_data = [team.to_dict(include_members=True) for team in teams]
return JsonResponse({'teams': teams_datal)
except Exception as e:
print(f'Get teams error: {e])
return JsonResponse(f'message': *Server error , status=500)

@csrf_exempt
@require_http_methodsC["GET"])
@authenticate
def get_team_by_id(request, team_id):
try:
try:
team = Team.objects.prefetch_related('members').get(id=team_id)
except Team.DoesNotExist:
return JsonResponse (f"message": "Team not found"}, status=484)
return JsonResponse({'team': team.to_dict(include_members=True)})
except Exception as e:
print(f'Get team error: {e}')
return JsonResponse(fmessage': "Server error'}, status=500)

@csrf_exempt
@require_http_methods(["POST"])

def create_team (request):
try:
try:
name = data.get(' name', '•)

key = data.get('key', "')

icon = data.get('icon', 'settings')

icon_color = data.get('iconColor*, None)

description = data.get('description', '')

members = data.get('members', [1)

data = json. Loads(request.body)
except JSONDecodeError:
return JsonResponse(f'message': 'Invalid JSON'}, status=400)
if not name or not key:
return JsonResponse(f"message': "Name and key are required"}, status=400)
if Team. objects.filter(key=key.upper)) .exists):
return JsonResponse(f'message': "Team key already exists"}, status=400)

team = Team. objects. create(
name-name, key=key. upper(),
Icon=icon,
icon_color=icon_color.
description=description,
if members:
team.members.set(members)
75
team = Team.objects.prefetch_related("members').get(id=team.1d)
return JsonResponse (1 message*: Team created successfully. "team": team. to_dict (incLude_members-True)}:status=201)
except Exception as e:
print(f' Create team error: {el)
return JsonResponse(f"message: "Server error"}, status=500)


@csrf_exempt
@require_http_methods(["PUT"])
def update_team(request, team_id):
try:
try:
data = json. Loads (request.body)
except JSONDecodeError:
return JsonResponse(f"message*: 'Invalid JSON'}, status=400)
team = Team.objects.get(id=team_1d)
if "key' in data and data[ key ].upper©) != team.key:
data[ key'] = datal key 1. upper()
return JsonResponse({message*: "Team updated successfully'. "team": team.to_dict incLudemembers-True)h)
except Exception as e:
print(f'Update team error: {e}')
return JsonResponse(l"message': "Server error*}, status=500)




@csrf_exempt
@require_http_methods(["DELETE"1)
def delete_team(request, team_id):
try:
return JsonResponse({"message': "Team deleted successfully'})
except Exception as e:
print(f'Delete team error: {e}')
return JsonResponse({message': 'Server error'}, status=500)
