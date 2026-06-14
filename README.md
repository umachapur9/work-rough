test_should_allow_admin_to_update_a_team
self= =<test.test_app.TestTeamManagement object at 0x7f4789074fb0>
def test_should_allow_admin_to_update_a_team(self):
# Admin updates team fields
response = self.client.put(
f'/api/teams/{self.test_team.id}',
data=json.dumps([
"name*: "Team CRUD Alpha Updated" l
'icon': 'rocket',
'iconColor': "blue'
content_type= appLication/json•
HTTP_AUTHORIZATION=f'Bearer {self.admin_token}'.
)
# Should return updated team details
assert response.status_code == 200
assert 500 == 200
E
+
where 500 = <JsonResponse status_code=500, "application/json">.st
test/test_app.py:96: AssertionError


test_should_prevent_non_admin_from_updating_a_team
self = stest.test_app. TestTeamManagement object at 0x7f4789075468>
def test_should_prevent_non_admin_from_updating_a_team(self):
# Member attempts to update team
response - self.client.put(
f'/api/teams/{self.test_team.id}',
data-json. dumps({"name": "Team CRUD Member Update"F), content_type-'application/json'
HTTP_AUTHORIZATION=f Bearer {self.member_token}',
E
E
)
# Should reject with admin-only message
assert response. status_code -- 403
assert 500 == 403
+ where 500 = <JsonResponse status_code-500, "application/json">.statı
test/test_app.py:119: AssertionError

test_should_allow_admin_to_delete_a_team
self= <test.test_app.TestTeamManagement object at 0x7f4789075a00>
def test_should_allow_admin_to_delete_a_team(self):
# Admin deletes a team
response = self.client.delete(
f'/api/teams/{self.test_team.id}',
HTTP_AUTHORIZATION=fBearer {self.admin_token},
# Should confirm delete and remove from DB
assert response.status_code == 200
assert 500 == 200
+
where 500= <JsonResponse <JsonResponse status_code=500,"application/json">.status_cade
test/test_app.py: 187: AssertionError
