# Implementation Guide for Enhanced Features

This guide provides technical instructions for implementing the enhanced features in the Romper Room Poker League website.

## Directory Structure

The enhanced features have been added to the following files:

```
romper-room-poker/
├── src/
│   ├── app/
│   │   ├── admin/
│   │   │   └── page.tsx       # Admin Dashboard implementation
│   │   └── page.tsx           # Main registration page
│   └── lib/
│       ├── firebase.js        # Firebase configuration
│       ├── firebaseService.js # Player registration service
│       ├── firebaseAuthService.js    # Authentication service
│       ├── firebaseStorageService.js # Profile photo storage service
│       └── firebaseEmailService.js   # Email notification service
```

## Installation Requirements

To fully implement these enhanced features, you'll need to install the following dependencies:

```bash
npm install firebase
```

## Firebase Configuration

The Firebase configuration is already set up in `src/lib/firebase.js` with your project credentials:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyBjiVPQYT3sCZP3oeGSFzMO5eeoqNHujoQ",
  authDomain: "rrpl-e1668.firebaseapp.com",
  projectId: "rrpl-e1668",
  storageBucket: "rrpl-e1668.firebasestorage.app",
  messagingSenderId: "1008241401798",
  appId: "1:1008241401798:web:670bf722ae1b0dd4c71cc7",
  measurementId: "G-4E0MY0499D"
};
```

## Firebase Services Setup

### 1. Authentication Setup

1. Go to the Firebase Console: https://console.firebase.google.com/
2. Select your project (rrpl-e1668)
3. In the left sidebar, click "Authentication"
4. Click "Get started"
5. Enable "Email/Password" authentication
6. Create an admin user:
   - Click "Add user"
   - Enter an email and password for the admin
   - Click "Add user"

### 2. Storage Setup

1. In the Firebase Console, click "Storage"
2. Click "Get started"
3. Choose "Start in production mode"
4. Select a location (same as your Firestore database)
5. Click "Next"
6. Update the security rules to:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /profile-photos/{fileName} {
      allow read: if true;
      allow write: if request.auth != null || request.resource.size < 5 * 1024 * 1024;
    }
  }
}
```

### 3. Email Service Setup

For production email sending, you'll need to:

1. Sign up for an email service provider (SendGrid, Mailgun, etc.)
2. Get API keys from your provider
3. Update the `firebaseEmailService.js` file to use the provider's API
4. Store API keys securely (environment variables)

## Integration with Main Registration Form

The main registration form (`src/app/page.tsx`) needs to be updated to use the new services:

1. Import the new services:

```javascript
import { uploadProfilePhoto } from '../lib/firebaseStorageService';
import { sendWelcomeEmail, sendAdminNotification } from '../lib/firebaseEmailService';
```

2. Update the form submission handler to use these services:

```javascript
const handleSubmit = async (e) => {
  e.preventDefault();
  
  if (validateForm()) {
    setIsSubmitting(true);
    setSubmitError('');
    
    try {
      // Generate a sequential player number
      const newPlayerNumber = await generatePlayerNumber();
      setPlayerNumber(newPlayerNumber);
      
      // Upload profile photo if provided
      let photoURL = null;
      if (formData.profilePhoto) {
        const uploadResult = await uploadProfilePhoto(formData.profilePhoto, newPlayerNumber);
        if (uploadResult.success) {
          photoURL = uploadResult.url;
        }
      }
      
      // Prepare data for Firebase
      const playerData = {
        firstName: formData.firstName,
        lastName: formData.lastName,
        email: formData.email,
        phone: formData.phone,
        address: {
          street: formData.street || '',
          city: formData.city || '',
          state: formData.state || '',
          zip: formData.zip || ''
        },
        birthday: formData.birthMonth && formData.birthDay && formData.birthYear 
          ? `${formData.birthMonth}/${formData.birthDay}/${formData.birthYear}`
          : '',
        referredBy: formData.referredBy || '',
        profilePhotoURL: photoURL,
        hasProfilePhoto: !!photoURL
      };
      
      // Save to Firebase
      const result = await savePlayerRegistration(playerData, newPlayerNumber);
      
      if (result.success) {
        // Send welcome email to player
        await sendWelcomeEmail(playerData, newPlayerNumber);
        
        // Send notification to admin
        await sendAdminNotification(playerData, newPlayerNumber);
        
        setIsSubmitted(true);
      } else {
        setSubmitError('There was an error saving your registration. Please try again.');
      }
    } catch (error) {
      console.error('Error during form submission:', error);
      setSubmitError('An unexpected error occurred. Please try again later.');
    } finally {
      setIsSubmitting(false);
    }
  }
};
```

## Accessing the Admin Dashboard

Once deployed, the Admin Dashboard will be available at:

```
https://www.romperroompoker.com/admin
```

You'll need to log in with the admin credentials you created in the Firebase Console.

## Security Considerations

1. **Firestore Rules**: Update your Firestore security rules to protect player data:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /players/{playerId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null || request.resource.data.keys().hasAll(['firstName', 'lastName', 'email', 'phone', 'playerNumber']);
    }
    match /emailLogs/{logId} {
      allow read, write: if request.auth != null;
    }
  }
}
```

2. **Environment Variables**: For production, store Firebase credentials in environment variables

3. **Regular Backups**: Set up regular backups of your Firestore database

## Testing the Enhanced Features

1. **Admin Dashboard**: Test logging in and viewing player data
2. **Profile Photos**: Test uploading profile photos during registration
3. **Email Notifications**: Test the email simulation in development
4. **Authentication**: Test admin login and logout functionality

## Deployment Steps

Follow the same deployment steps as outlined in the main deployment documentation, ensuring that:

1. All Firebase services are properly configured
2. All dependencies are installed
3. The website is built with the enhanced features
4. The domain is properly connected

## Troubleshooting

If you encounter issues with the enhanced features:

1. **Firebase Authentication Issues**:
   - Check Firebase Console for authentication errors
   - Verify that Email/Password authentication is enabled

2. **Storage Upload Issues**:
   - Check storage rules in Firebase Console
   - Verify that the storage bucket is properly initialized

3. **Email Sending Issues**:
   - Check email service provider dashboard for errors
   - Verify API keys and configuration

4. **Admin Dashboard Access Issues**:
   - Clear browser cache and cookies
   - Verify admin user credentials in Firebase Console
