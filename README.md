<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Password Strength Tester</title>
    <!-- Load Tailwind CSS for easy styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Load Inter font -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6; /* Light gray background */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh; /* Full viewport height */
            margin: 0;
            padding: 1rem; /* Add some padding for smaller screens */
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <div class="bg-white p-8 rounded-lg shadow-xl w-full max-w-md">
        <h1 class="text-3xl font-bold text-center text-gray-800 mb-6">Password Strength Tester</h1>

        <div class="mb-6">
            <label for="passwordInput" class="block text-gray-700 text-sm font-semibold mb-2">Enter your password:</label>
            <input
                type="password"
                id="passwordInput"
                class="shadow-sm appearance-none border rounded-lg w-full py-3 px-4 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200"
                placeholder="Type your password here..."
                aria-label="Password Input"
            >
        </div>

        <!-- Password Strength Feedback -->
        <div class="mb-4">
            <p class="text-sm font-semibold text-gray-700 mb-2">Strength:</p>
            <div id="strengthFeedback" class="py-2 px-4 rounded-lg text-center font-bold text-white transition-all duration-300 ease-in-out">
                No password entered
            </div>
        </div>

        <!-- Suggestions for Improvement -->
        <div>
            <p class="text-sm font-semibold text-gray-700 mb-2">Suggestions:</p>
            <ul id="suggestionsList" class="list-disc list-inside text-gray-600 text-sm">
                <li>Start typing to get feedback.</li>
            </ul>
        </div>
    </div>

    <script>
        // Get references to HTML elements
        const passwordInput = document.getElementById('passwordInput');
        const strengthFeedback = document.getElementById('strengthFeedback');
        const suggestionsList = document.getElementById('suggestionsList');

        // Add an event listener to the password input field
        passwordInput.addEventListener('input', checkPasswordStrength);

        /**
         * Checks the strength of the entered password and updates the UI.
         */
        function checkPasswordStrength() {
            const password = passwordInput.value;
            let score = 0; // Initialize strength score
            const suggestions = []; // Array to store suggestions

            // Reset feedback styles
            strengthFeedback.className = 'py-2 px-4 rounded-lg text-center font-bold text-white transition-all duration-300 ease-in-out';
            strengthFeedback.textContent = 'No password entered';
            suggestionsList.innerHTML = ''; // Clear previous suggestions

            if (password.length === 0) {
                suggestionsList.innerHTML = '<li>Start typing to get feedback.</li>';
                return; // Exit if no password is entered
            }

            // --- Scoring Logic ---

            // 1. Length
            if (password.length >= 8) {
                score += 1; // Good length
                if (password.length >= 12) {
                    score += 1; // Better length
                }
                if (password.length >= 16) {
                    score += 1; // Excellent length
                }
            } else {
                suggestions.push('Password should be at least 8 characters long.');
            }

            // 2. Character Types
            const hasLowercase = /[a-z]/.test(password);
            const hasUppercase = /[A-Z]/.test(password);
            const hasNumber = /[0-9]/.test(password);
            const hasSpecialChar = /[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?~]/.test(password);

            let charTypeCount = 0;
            if (hasLowercase) { score += 1; charTypeCount++; } else { suggestions.push('Include lowercase letters.'); }
            if (hasUppercase) { score += 1; charTypeCount++; } else { suggestions.push('Include uppercase letters.'); }
            if (hasNumber) { score += 1; charTypeCount++; } else { suggestions.push('Include numbers.'); }
            if (hasSpecialChar) { score += 1; charTypeCount++; } else { suggestions.push('Include special characters (e.g., !@#$%^&*).'); }

            // Bonus for variety
            if (charTypeCount >= 3) {
                score += 1;
            }
            if (charTypeCount === 4) {
                score += 1;
            }

            // 3. Common Patterns / Weaknesses (Deductions)
            // Simple check for common weak passwords
            const commonWeakPasswords = ['password', '123456', 'qwerty', 'admin', '12345678', '123456789', 'password123'];
            if (commonWeakPasswords.includes(password.toLowerCase())) {
                score = 0; // Immediately make it very weak
                suggestions.push('Avoid common and easily guessable passwords.');
            }

            // Check for repeated characters (e.g., "aaaaa")
            if (/(.)\1{2,}/.test(password)) { // Matches 3 or more of the same character in a row
                score -= 1;
                suggestions.push('Avoid repeating characters excessively.');
            }

            // Check for sequential characters (e.g., "abcde", "12345")
            const sequentialPatterns = ['abc', '123', 'qwe', 'asd', 'zxc'];
            for (const pattern of sequentialPatterns) {
                if (password.toLowerCase().includes(pattern) || password.toLowerCase().includes(pattern.split('').reverse().join(''))) {
                    score -= 1;
                    suggestions.push('Avoid common sequential patterns (e.g., "abc", "123").');
                    break; // Only suggest once
                }
            }

            // --- Update UI based on Score ---
            let strengthText = '';
            let bgColorClass = '';

            if (score <= 2) {
                strengthText = 'Very Weak';
                bgColorClass = 'bg-red-500';
            } else if (score <= 4) {
                strengthText = 'Weak';
                bgColorClass = 'bg-orange-500';
            } else if (score <= 6) {
                strengthText = 'Moderate';
                bgColorClass = 'bg-yellow-500';
            } else if (score <= 8) {
                strengthText = 'Strong';
                bgColorClass = 'bg-green-500';
            } else {
                strengthText = 'Very Strong';
                bgColorClass = 'bg-green-700';
            }

            // Update strength feedback text and color
            strengthFeedback.textContent = strengthText;
            strengthFeedback.classList.add(bgColorClass);

            // Update suggestions list
            if (suggestions.length === 0 && password.length > 0) {
                suggestionsList.innerHTML = '<li>Great job! Your password looks strong.</li>';
            } else {
                suggestions.forEach(suggestion => {
                    const li = document.createElement('li');
                    li.textContent = suggestion;
                    suggestionsList.appendChild(li);
                });
            }
        }
    </script>
</body>
</html>
