# URL-Based QR Code Verification System

## Overview
The PQRSTU attendance system now uses URL-based QR codes that redirect users to a verification page. When students scan the QR code with their mobile devices, they are automatically verified and their attendance is marked.

## How It Works

### 1. **Faculty Generates Dynamic QR Code**
When a faculty member starts an attendance session:
- A unique session secret key (`kv_key`) is generated
- Every 15 seconds (configurable), a new QR code is created
- Each QR code contains:
  - **Token**: Base64-encoded payload with session ID, expiry time, and nonce
  - **Signature**: HMAC-SHA256 signature of the token using the session secret

### 2. **QR Code URL Format**
```
https://indrasenareddy11v.github.io/verify.html?token=<BASE64_TOKEN>&sig=<HMAC_SIGNATURE>
```

### 3. **Student Scans QR Code**
When a student scans the QR code from their mobile device:
1. The device's camera app or QR scanner detects the URL
2. The URL automatically opens in the browser
3. The browser loads `verify.html` with the token and signature

### 4. **Automatic Verification Process**
The `verify.html` page performs the following checks:

#### a. **Token Validation**
- Decodes the Base64 token
- Checks if the QR code has expired
- Verifies the token version

#### b. **Session Validation**
- Fetches the attendance session from Firebase
- Confirms the session is still "live"
- Retrieves the session's secret key

#### c. **Signature Verification**
- Recomputes the HMAC signature using the session secret
- Compares it with the signature from the QR code
- **If signatures don't match → FAIL**

#### d. **Authentication Check**
- Checks if the user is signed in to Firebase
- **If not signed in** → Redirects to `student.html` with return URL
- After login, automatically returns to verification page

#### e. **Profile Validation**
- Ensures the user has completed their student profile
- **If no profile** → Shows error

#### f. **Duplicate Check**
- Checks if attendance already marked for this user + session
- Checks if attendance already marked from this device
- **If duplicate** → Shows success (already marked)

#### g. **Geolocation Verification**
- Captures the student's GPS coordinates
- Calculates distance from classroom location
- **If too far** → Shows "Location verification failed" error

#### h. **Mark Attendance**
- If all checks pass, creates a checkin record in Firebase
- Shows **"Congrats! Your attendance has been marked successfully!"**

### 5. **Success/Failure Messages**

#### Success Scenarios:
✅ **"Congrats! Your attendance has been marked successfully!"**
- All validations passed
- Attendance recorded in database
- Shows course name

✅ **"Attendance already marked!"**
- User already marked attendance for this session
- Prevents duplicate entries

#### Failure Scenarios:
❌ **"Invalid QR code"**
- Token missing or malformed
- Could not decode token

❌ **"QR code expired"**
- The QR code has passed its expiry time
- Student should scan the latest QR

❌ **"Invalid signature"**
- Token was tampered with
- Signature verification failed

❌ **"Session ended"**
- The faculty has ended the attendance session
- No longer accepting attendance

❌ **"Location verification failed"**
- Student is too far from the classroom
- Shows distance and required radius

❌ **"Already marked from this device"**
- Prevents proxy attendance
- Different user can't mark from same device

## Security Features

### 1. **HMAC Signature**
- Each QR code is cryptographically signed
- Prevents forgery or tampering
- Secret key never leaves the server

### 2. **Time-Based Expiry**
- QR codes expire after 90 seconds (configurable)
- Old QR codes cannot be reused
- Rotating QR codes every 15 seconds

### 3. **GPS Geofencing**
- Validates student is physically present in classroom
- Configurable radius (default 60 meters)
- High accuracy GPS required

### 4. **Device Fingerprinting**
- Each device has a unique ID
- Prevents multiple students from using same device
- Stored in localStorage

### 5. **Firebase Authentication**
- Students must be authenticated
- Email verification required
- Profile completion mandatory

## Deployment

### Your Domain
The QR codes are configured to use:
```
https://indrasenareddy11v.github.io/verify.html
```

### Required Files
1. **verify.html** - Verification page (main entry point)
2. **student.html** - Login/profile page (with redirect support)
3. **faculty.html** - Faculty control panel (generates QR codes)

### Firebase Configuration
Ensure Firebase Firestore has the following collections:
- `slots` - Attendance sessions
- `users` - Student profiles
- `checkins` - Attendance records
- `raise_tokens` - Exception tokens

## Mobile User Experience

1. **Student opens camera app**
2. **Points camera at faculty's QR code on screen**
3. **Notification appears: "Open in Safari/Chrome"**
4. **Taps notification**
5. **Browser opens verify.html**
6. **Sees loading spinner: "Verifying your attendance..."**
7. **If not logged in**: Redirected to login → Returns after login
8. **If all checks pass**: Green success message ✅
9. **If something fails**: Red error message ❌ with reason

## Testing

### Test Success Flow:
1. Faculty starts session
2. Student is signed in
3. Student is within classroom radius
4. Student scans QR within 90 seconds
5. Should see: "Congrats! Your attendance has been marked successfully!"

### Test Failure Flows:
1. **Expired QR**: Wait 90+ seconds, then scan
2. **Wrong location**: Be >60m away from classroom
3. **No login**: Sign out, then scan QR
4. **Duplicate**: Scan QR twice
5. **Tampered QR**: Manually edit URL parameters

## Troubleshooting

### "Invalid QR code"
- QR might be corrupted
- URL parameters missing
- Try scanning again

### "QR code expired"
- QR rotates every 15 seconds
- Scan the latest QR on faculty's screen

### "Location verification failed"
- Enable GPS/Location services
- Make sure you're in the classroom
- Check signal strength

### "Session not found"
- Faculty may have ended the session
- Check with faculty

### Not redirecting after login
- Check browser allows redirects
- Clear cache and try again
- Check URL parameters preserved

## Future Enhancements

1. **Bluetooth verification** - Additional proximity check
2. **Face recognition** - Anti-proxy via camera
3. **Push notifications** - Alert when session starts
4. **Offline QR caching** - Mark attendance when offline
5. **Analytics dashboard** - Attendance patterns and insights

---

**Developed by Team 8 for TERI SAS**  
*Project QR for Students of TERI University (PQRSTU)*
