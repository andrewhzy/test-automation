I need to create a test-automation service, for testing glean chat api.

current status:
- we have 20 users to do testing after we make change to glean service( e.g. upgrade a model).
- we will prepare 30 questions. 
- each user open the glean UI to chat, post the 30 questions one by one, collect answers and past into an excel file in the answer column(the excel file will have at least 2 columns: question, answer).

wanted status:
- create a service to do test automatically for the 20 users
- only 4 user are alowed to trigger the test, because the 20 users are not tech background, and we don't want to provide a UI for them to test by themselves
- only 30 questions are allowed to be tested
- these 4 test-request users, 20 test users, 30 questions will be configurable.

## first pls help to design an architectuer:
- include some high level components: client, sso svc, test-auto svc, glean svc.
- when test-auto recieves a request, it will:
    1. the request will include a list test users, a list of questions.
    2. authentication
    3. authorization:
        1. incoming reqest user is in an allowlist of test-request users
        2, the test user list all in a test users allow list.
        3. question list all allowed.
    4. then loop through test users.
    5. then loop through questions.
    6. call glean chat api
    7. collect all answers for all test users, then return to the client.

the architecture.md need update:
- after request is authorized, when looping through all test users, the app will use a GLEAN_API_TOKEN and act as the test user to call glean's /rest/api/v1/createauthtoken, this will get a glean_auth_token for the test user, then for each question, the service will call glean's chat api with the question and pass the glean_auth_token.
- pls go through the architecture.md file and udpate accordingly.

help to update oas.yaml:
1. I only need 3 endpoints:
   1. POST /testautomation/tests
   2. GET /testautomation/allowconfig
   3. GET /healthcheck
pls consult test-automation-architecture.md. before updating     
