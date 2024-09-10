# RegistrationForm

Frontend (HTML & CSS)

<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Registration Form</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        .container {
            background-color: white;
            padding: 40px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            width: 100%;
            max-width: 400px;
        }

        .container h2 {
            text-align: center;
            margin-bottom: 20px;
            font-size: 24px;
        }

        .form-group {
            margin-bottom: 15px;
        }

        .form-group label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }

        .form-group input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        .form-group input[type="submit"] {
            background-color: #333;
            color: white;
            cursor: pointer;
            border: none;
        }

        .form-group input[type="submit"]:hover {
            background-color: #555;
        }

        .message {
            margin-top: 20px;
            text-align: center;
            color: green;
        }

        .error {
            color: red;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>Sign Up</h2>
    <form id="registrationForm">
        <div class="form-group">
            <label for="name">Full Name</label>
            <input type="text" id="name" placeholder="Enter your name" required>
        </div>
        <div class="form-group">
            <label for="email">Email</label>
            <input type="email" id="email" placeholder="Enter your email" required>
        </div>
        <div class="form-group">
            <label for="password">Password</label>
            <input type="password" id="password" placeholder="Enter your password" required>
        </div>
        <div class="form-group">
            <label for="confirmPassword">Confirm Password</label>
            <input type="password" id="confirmPassword" placeholder="Confirm your password" required>
        </div>
        <div class="form-group">
            <input type="submit" value="Register">
        </div>
    </form>
    <div class="message" id="message"></div>
</div>

<script>
    document.getElementById('registrationForm').addEventListener('submit', async function (event) {
        event.preventDefault();

        const name = document.getElementById('name').value;
        const email = document.getElementById('email').value;
        const password = document.getElementById('password').value;
        const confirmPassword = document.getElementById('confirmPassword').value;
        const messageDiv = document.getElementById('message');

        // Validate if passwords match
        if (password !== confirmPassword) {
            messageDiv.innerHTML = "<p class='error'>Passwords do not match.</p>";
            return;
        }

        // Send data to the server using fetch API
        try {
            const response = await fetch('/register', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    name: name,
                    email: email,
                    password: password,
                }),
            });

            const data = await response.json();
            if (response.ok) {
                messageDiv.innerHTML = "<p>Registration successful!</p>";
                document.getElementById('registrationForm').reset();
            } else {
                messageDiv.innerHTML = `<p class='error'>${data.message}</p>`;
            }
        } catch (error) {
            messageDiv.innerHTML = "<p class='error'>An error occurred. Please try again.</p>";
        }
    });
</script>

</body>
</html>


Backend (Node.js + Express)
1.Initialize a Node.js project:
npm init -y

2.Install necessary packages:
npm install express mongoose body-parser bcryptjs cors

3.Create a new file called server.js and set up your Node.js server:
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const userRoutes = require('./routes/userRoutes');
const path = require('path');

const app = express();

// Middleware
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static(path.join(__dirname, 'public')));

// MongoDB connection (replace with your MongoDB URI)
mongoose.connect('mongodb://localhost:27017/registrationDB', {
    useNewUrlParser: true,
    useUnifiedTopology: true
}).then(() => {
    console.log("Connected to MongoDB");
}).catch(err => {
    console.error('Error connecting to MongoDB:', err.message);
});

// Routes
app.use(userRoutes);

// Server setup
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});

4.Create User Model
Create the MongoDB user model in models/userModel.js:
const mongoose = require('mongoose');

// Create a schema for user registration
const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true
    },
    email: {
        type: String,
        required: true,
        unique: true
    },
    password: {
        type: String,
        required: true
    }
});

// Create the User model
const User = mongoose.model('User', userSchema);

module.exports = User;

5.User Routes
Create the routes for handling user registration in routes/userRoutes.js:

const express = require('express');
const router = express.Router();
const User = require('../models/userModel');

// Registration route
router.post('/register', async (req, res) => {
    const { name, email, password } = req.body;

    try {
        // Check if the user already exists
        const existingUser = await User.findOne({ email });
        if (existingUser) {
            return res.send('<p>User already exists</p>');
        }

        // Create new user
        const newUser = new User({
            name,
            email,
            password
        });

        await newUser.save();
        res.send('<p>Registration successful!</p>');
    } catch (err) {
        console.error(err);
        res.send('<p>Error occurred during registration.</p>');
    }
});

module.exports = router;


6. Running the App
I)Start MongoDB
II)Run the server:
node server.js
