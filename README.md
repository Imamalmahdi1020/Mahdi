<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login and Sign Up</title>
    <style>
        form {
            display: flex;
            flex-direction: column;
            width: 300px;
            margin: auto;
        }
        input {
            margin: 10px 0;
        }
        #message {
            color: red;
        }
    </style>
</head>
<body>
    <h2>Sign Up</h2>
    <form id="signup-form">
        <input type="email" id="signup-email" placeholder="Email" required>
        <input type="password" id="signup-password" placeholder="Password" required>
        <button type="submit">Sign Up</button>
    </form>

    <h2>Login</h2>
    <form id="login-form">
        <input type="email" id="login-email" placeholder="Email" required>
        <input type="password" id="login-password" placeholder="Password" required>
        <button type="submit">Login</button>
    </form>

    <p id="message"></p>

    <script src="https://apis.google.com/js/api.js"></script>
    <script>
        const CLIENT_ID = 'YOUR_CLIENT_ID';
        const API_KEY = 'YOUR_API_KEY';
        const DISCOVERY_DOCS = ["https://sheets.googleapis.com/$discovery/rest?version=v4"];
        const SCOPES = "https://www.googleapis.com/auth/spreadsheets";

        function handleClientLoad() {
            gapi.load('client:auth2', initClient);
        }

        function initClient() {
            gapi.client.init({
                apiKey: API_KEY,
                clientId: CLIENT_ID,
                discoveryDocs: DISCOVERY_DOCS,
                scope: SCOPES
            }).then(function () {
                // Listen for sign-in state changes.
                gapi.auth2.getAuthInstance().isSignedIn.listen(updateSigninStatus);
                // Handle the initial sign-in state.
                updateSigninStatus(gapi.auth2.getAuthInstance().isSignedIn.get());
            }, function(error) {
                console.log(JSON.stringify(error, null, 2));
            });
        }

        function updateSigninStatus(isSignedIn) {
            if (isSignedIn) {
                console.log('User signed in');
            } else {
                gapi.auth2.getAuthInstance().signIn();
            }
        }

        function appendDataToSheet(email, password) {
            const params = {
                spreadsheetId: 'YOUR_SPREADSHEET_ID',
                range: 'Sheet1!A:B',
                valueInputOption: 'USER_ENTERED',
            };

            const valueRangeBody = {
                "values": [
                    [email, password]
                ]
            };

            gapi.client.sheets.spreadsheets.values.append(params, valueRangeBody).then((response) => {
                console.log(response.result);
                document.getElementById('message').textContent = 'Sign up successful!';
            }, (error) => {
                console.error(error.result.error.message);
            });
        }

        document.getElementById('signup-form').addEventListener('submit', function (event) {
            event.preventDefault();
            const email = document.getElementById('signup-email').value;
            const password = document.getElementById('signup-password').value;

            appendDataToSheet(email, password);
        });

        document.getElementById('login-form').addEventListener('submit', function (event) {
            event.preventDefault();
            const email = document.getElementById('login-email').value;
            const password = document.getElementById('login-password').value;

            // Verify the email and password
            const params = {
                spreadsheetId: 'YOUR_SPREADSHEET_ID',
                range: 'Sheet1!A:B',
            };

            gapi.client.sheets.spreadsheets.values.get(params).then((response) => {
                const users = response.result.values;
                const user = users.find(user => user[0] === email && user[1] === password);
                if (user) {
                    document.getElementById('message').textContent = 'Login successful!';
                } else {
                    document.getElementById('message').textContent = 'Invalid email or password.';
                }
            }, (error) => {
                console.error(error.result.error.message);
            });
        });

        handleClientLoad();
    </script>
</body>
</html>
