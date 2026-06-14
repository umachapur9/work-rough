test/test_app.py::TestTeam/anagement:: test_should_allow_admin_to_delete_a_team
FAILED [180%]
FAILURES =
TestTeanilanagement. test_should_allow_admin_to_update_a_team
self= ctest.test_app.TestTeamManagement object at 0x7f07db1f0c50>
def test_should_allow_admin_to_update_a_team(self):
# Admin updates team fields
response = self.client.put(
f*/api/teams/{self.test_team.id)',
data=json.dumps(f
"name": "Team CRUD Alpha Updated',
"icon": "rocket',
"IconColor"; "blue',
content_type='appLication/Json',
HTTPAUTHORIZATION=f"Bearer (self, admin_token)',
# Should return updated team details

assert response.status_code == 200
assert 508
200
+ where 500= <JsonResponse statuscode-500, "appLication/json">.status_code
test/test_app-py:96: AssertionError
-- Captured stdout call --
Update team error: "User" object has no attribute 'get'
-- Captured Log call ---
ERROR
django.request:Log.py:246 Internal Server Error: /api/teams/1
- TestTeamflanagement.test_should_prevent_.non_admin_from_updating_a_team
self = stest.test app.TestTeamManagement object at 8x7f07dad358ed>
def test_should_prevent_non_admin_from_updating_a_team(self):
# Member attempts to update team
response = self.client.put(
*'/api/teams/(self.test_team, 1d)',
data=json.dumpps(f'name"; "Team CRUD Member Update'', content_type="appLication/Json',
HTTP_AUTHORIZATION=f'Bearer (self,member_token)',

HTTPAUTHORIZATION=fBearer (self.member_token)',
vS.py
riews.py
s.py
s.py
views.py s.py
e.py
9+
backend lite3
y
)
>
E
E
# Should reject with admin-only message
assert response.status_code == 403
assert
500
405
where 500 = <JsonResponse status_code-590, "appLication/json">.status_code
test/test_app-py:119: AssertionError
-- Captured stdout call --
Update team error: "User" object has no attribute "get"
--- Captured Log call ----
ERROR
django.request:log.py:246 Internal Server Error: /api/teams/3
_ TestTeamanagement.test_should_prevent_non_admin_from_deleting_a_team _
self= ctest.test_app.TestTeamManagement object at 8x7f07dad35dc0>
def test_should_prevent_non_admin_from_deleting_a_team(self):
# Member attempts to delete team
response = self.client.deleteC
**/ap1/teams/(self.test_team. id)',
HTTP_AUTHORIZATION=fBearer (self-member_token)',
ents.txt
dules
)
# Should reject with admin-only message'

>
E
# Should confirm delete and remove from DB
assert response.status_code == 200
assert 500 == 200
+ where 500 = <JsonResponse status_code=500, "appLication/json">.status_code
test/test_app-py:187: AssertionError
- Captured stdout call
Delete team error: name "requests' is not defined
Captured Log call -
ERROR
django.request:Log.py:246 Internal Server Error: /api/teams/11
----- generated xml file: /projects/challenge/output/Team.xml -----
• (ven) user/projects/challenge $

- short test summary info -

FAILED test/test_app.py: :TestTeamManagement:: test_should_allow_admin_to_update_a_team - assert 500 = 200

* where 500 = «JsonResponse status_code-500, "appLication/json">.status_code

FAILED test/test_app.py: :TestTeam/anagement:: test_should_prevent_non_admin_ from_updating_a_team - assert 500 = 403

+ where 508 = JsonResponse status_code=500, "appLication/json">. status_code

FAILED test/test_app-py::TestTeam/anagement:: test_should_prevent_non_admin_fromdeleting_a_team - assert 500 == 403

* where 509 = JsonResponse status_code=500, "appLication/json">.status_code

FAILED test/test_app-py::TestTeamManagement:: test_should reject_update_when_key_already_exists - assert 500 == 400

* where 500 = JsonResponse status_code=500, "appLication/json">.status_code

FAILED test/test_app.py: :TestTeam/anagement::test_should_return_484_when_updating_or_deleting_non existent_team - as
sert 580 == 404


* where 500 = <JsonResponse status_code=500, "appLication/json">. status_code

FATLED test/test_app.py: :TestTeam/anagement:: test_should_allow_admin_to_delete_a_team - assert 500 = 200

+ where 508 = <JsonResponse status_code=500, "appLication/json">, status_code

: 6 falled in 7.88s =
