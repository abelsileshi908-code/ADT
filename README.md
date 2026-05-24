<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Abel Digital Tekemach</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        body { background-color: #0b0f19; color: #ffffff; display: flex; justify-content: center; align-items: center; min-height: 100vh; padding: 20px; }
        .app-container { width: 100%; max-width: 400px; background: #131a26; border-radius: 24px; padding: 24px; box-shadow: 0 12px 40px rgba(0,0,0,0.5); border: 1px solid #1f2c3f; }
        .header { text-align: center; margin-bottom: 24px; }
        .header h1 { font-size: 22px; color: #38bdf8; letter-spacing: 1px; }
        .screen { display: none; }
        .active-screen { display: block; }
        
        /* Form Inputs */
        .input-group { margin-bottom: 16px; }
        .input-group label { display: block; font-size: 12px; color: #94a3b8; margin-bottom: 6px; text-transform: uppercase; }
        .input-group input { width: 100%; padding: 14px; background: #1e293b; border: 1px solid #334155; border-radius: 12px; color: white; font-size: 16px; outline: none; transition: 0.3s; }
        .input-group input:focus { border-color: #38bdf8; }
        
        /* Buttons */
        .btn { width: 100%; padding: 14px; background: #38bdf8; border: none; border-radius: 12px; color: #0b0f19; font-weight: bold; font-size: 16px; cursor: pointer; transition: 0.2s; }
        .btn:hover { background: #0ea5e9; }
        .error-msg { color: #f87171; font-size: 14px; text-align: center; margin-top: 10px; display: none; }

        /* Dashboard Elements */
        .welcome-text { font-size: 16px; color: #94a3b8; margin-bottom: 12px; }
        .balance-card { background: linear-gradient(135deg, #1e293b 0%, #0f172a 100%); padding: 24px; border-radius: 16px; border: 1px solid #334155; text-align: center; margin-bottom: 20px; position: relative; overflow: hidden; }
        .balance-card::before { content: ''; position: absolute; top: -50%; left: -50%; width: 200%; height: 200%; background: radial-gradient(circle, rgba(56,189,248,0.05) 0%, transparent 70%); }
        .balance-label { font-size: 12px; color: #38bdf8; text-transform: uppercase; letter-spacing: 1px; margin-bottom: 4px; }
        .balance-amount { font-size: 32px; font-weight: bold; color: #ffffff; }
        
        .badge { display: block; text-align: center; padding: 10px; border-radius: 8px; font-size: 13px; font-weight: 600; margin-bottom: 16px; background: #1e293b; border: 1px solid #334155; }
        
        .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
        .stat-box { background: #1e293b; padding: 14px; border-radius: 12px; border: 1px solid #334155; }
        .stat-label { font-size: 11px; color: #94a3b8; text-transform: uppercase; margin-bottom: 4px; }
        .stat-value { font-size: 16px; font-weight: bold; color: #f1f5f9; }
    </style>
</head>
<body>

<div class="app-container">
    <div class="header">
        <h1>◢◤ DIGITAL TEKEMACH ◢◤</h1>
    </div>

    <!-- LOGIN SCREEN -->
    <div id="loginScreen" class="screen active-screen">
        <div class="input-group">
            <label>Member ID NO</label>
            <input type="text" id="memberIdInput" placeholder="e.g. DT-001">
        </div>
        <div class="input-group" id="pinGroup" style="display:none;">
            <label>Security PIN</label>
            <input type="password" id="pinInput" placeholder="4-Digit PIN" maxlength="4" inputmode="numeric">
        </div>
        <button class="btn" id="loginBtn" onclick="handleLoginSubmit()">Continue</button>
        <div class="error-msg" id="loginError">Incorrect details. Please try again.</div>
    </div>

    <!-- REGISTRATION SCREEN (FOR SETTING OWN PIN) -->
    <div id="registerScreen" class="screen">
        <p style="font-size:14px; color:#94a3b8; margin-bottom:15px; text-align:center;">Welcome! Set your custom 4-digit security PIN to lock your account balance.</p>
        <div class="input-group">
            <label>Create 4-Digit PIN</label>
            <input type="password" id="newPin" maxlength="4" inputmode="numeric">
        </div>
        <div class="input-group">
            <label>Confirm PIN</label>
            <input type="password" id="confirmPin" maxlength="4" inputmode="numeric">
        </div>
        <button class="btn" onclick="saveNewPin()">Secure Account</button>
        <div class="error-msg" id="registerError">PINs do not match.</div>
    </div>

    <!-- MAIN DASHBOARD -->
    <div id="dashboardScreen" class="screen">
        <div class="welcome-text" id="welcomeUser">Welcome, Member</div>
        
        <!-- Big Total Balance Card -->
        <div class="balance-card">
            <div class="balance-label">Total Saved So Far</div>
            <div class="balance-amount" id="userBalance">0.00 ETB</div>
        </div>

        <!-- Dynamic Fee / Invite Badge -->
        <div class="badge" id="feeBadge" style="color: #fbbf24;">
            Fee: 3% Active
        </div>

        <!-- Sheet Metrics Display -->
        <div class="stats-grid">
            <div class="stat-box">
                <div class="stat-label">Final Payout</div>
                <div class="stat-value" id="userPayout">0.00 ETB</div>
            </div>
            <div class="stat-box">
                <div class="stat-label">Days Remaining</div>
                <div class="stat-value" id="userDays">0 Days</div>
            </div>
            <div class="stat-box" style="grid-column: span 2;">
                <div class="stat-label">Savings Target Progress</div>
                <div class="stat-value" id="userProgress" style="color:#38bdf8;">0% Complete</div>
            </div>
        </div>
    </div>
</div>

<script>
    // ⚠️ REPLACE THIS WITH YOUR DEPLOYED GOOGLE APPS SCRIPT WEB APP URL
    const API_URL = "YOUR_DEPLOYED_WEB_APP_URL_HERE";
    
    let currentId = "";
    let step = 1;

    function handleLoginSubmit() {
        currentId = document.getElementById("memberIdInput").value.trim();
        const pin = document.getElementById("pinInput").value.trim();
        const errorDiv = document.getElementById("loginError");
        
        if (!currentId) return;

        errorDiv.style.display = "none";

        if (step === 1) {
            // First step: Check if user exists and needs register or login
            fetch(`${API_URL}?id=${currentId}&pin=CHECK`)
                .then(res => res.json())
                .then(data => {
                    if (data.status === "NOT_FOUND") {
                        errorDiv.innerText = "Member ID not found.";
                        errorDiv.style.display = "block";
                    } else if (data.status === "NEW_USER") {
                        showScreen("registerScreen");
                    } else {
                        // User exists and has a PIN, show PIN input box
                        document.getElementById("pinGroup").style.display = "block";
                        document.getElementById("loginBtn").innerText = "Login";
                        step = 2;
                    }
                });
        } else if (step === 2) {
            // Second step: Verify PIN
            fetch(`${API_URL}?id=${currentId}&pin=${pin}`)
                .then(res => res.json())
                .then(data => {
                    if (data.status === "SUCCESS") {
                        document.getElementById("welcomeUser").innerText = `⚡ Welcome, ${data.name}`;
                        document.getElementById("userBalance").innerText = `${Number(data.saved).toLocaleString()} ETB`;
                        document.getElementById("userPayout").innerText = `${Number(data.payout).toLocaleString()} ETB`;
                        document.getElementById("userDays").innerText = `${data.daysLeft} Days`;
                        
                        // Handle % formatting safely
                        let progressVal = typeof data.progress === 'number' ? (data.progress * 100).toFixed(0) : data.progress;
                        document.getElementById("userProgress").innerText = `${progressVal}% Complete`;
                        
                        // Dynamic text badge logic for referral system or fee status
                        const badge = document.getElementById("feeBadge");
                        if (data.referrals >= 5) {
                            badge.innerText = "🎉 VIP Status: 1% Fee Locked!";
                            badge.style.color = "#34d399";
                        } else {
                            let missing = 5 - (data.referrals || 0);
                            badge.innerText = `⚠️ Standard Fee (3%). Invite ${missing} more friends to unlock 1%!`;
                            badge.style.color = "#fbbf24";
                        }

                        showScreen("dashboardScreen");
                    } else {
                        errorDiv.innerText = "Incorrect security PIN.";
                        errorDiv.style.display = "block";
                    }
                });
        }
    }

    function saveNewPin() {
        const pin1 = document.getElementById("newPin").value.trim();
        const pin2 = document.getElementById("confirmPin").value.trim();
        const regError = document.getElementById("registerError");

        if(pin1.length !== 4 || isNaN(pin1)) {
            regError.innerText = "PIN must be a 4-digit number.";
            regError.style.display = "block";
            return;
        }

        if (pin1 !== pin2) {
            regError.innerText = "PINs do not match.";
            regError.style.display = "block";
            return;
        }

        fetch(API_URL, {
            method: "POST",
            body: JSON.stringify({ id: currentId, pin: pin1 })
        })
        .then(res => res.json())
        .then(data => {
            if (data.status === "PIN_SAVED") {
                alert("Security PIN saved successfully! Please log in now.");
                step = 1;
                document.getElementById("pinGroup").style.display = "none";
                document.getElementById("loginBtn").innerText = "Continue";
                showScreen("loginScreen");
            }
        });
    }

    function showScreen(screenId) {
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active-screen'));
        document.getElementById(screenId).classList.add('active-screen');
    }
</script>
</body>
</html>
