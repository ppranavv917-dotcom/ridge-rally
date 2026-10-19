# Ridge Rally: turn on the public friends server (about 5 minutes, free)

You do this ONCE. After that every player can pick a name, search names and add friends.

1. Go to https://console.firebase.google.com and sign in. Press **Add project**, name it `ridge-rally`, skip Google Analytics.
2. Left menu **Build > Authentication > Get started > Sign-in method**. Turn ON **Anonymous**. Save.
3. Left menu **Build > Realtime Database > Create database**. Pick a location, choose **Start in locked mode**.
4. Open the **Rules** tab, delete everything, paste the rules below, press **Publish**.
5. Press the gear icon > **Project settings > General**. Under "Your apps" press the `</>` (web) icon, register an app, and copy the `apiKey`. Also copy the **databaseURL** from the Realtime Database page (looks like `https://ridge-rally-default-rtdb.firebaseio.com`).
6. Open `config.js` in the site folder and paste both values. Upload the folder again (Netlify: drag it in again).
   For the single `ridge-rally.html` file, you can instead edit the line near the top that says `window.RR_CONFIG=...`.
7. In the Firebase console go to **Authentication > Settings > Authorized domains** and add your website domain.

The apiKey is safe to be public. The rules below are what protect the data.

## Rules (paste into the Rules tab)

```json
{
  "rules": {
    "users": {
      ".read": "auth != null",
      ".indexOn": ["nameLower"],
      "$u": {
        ".write": "auth != null && auth.uid == $u",
        ".validate": "newData.hasChildren(['name','nameLower','seen']) && newData.child('name').isString() && newData.child('name').val().length >= 3 && newData.child('name').val().length <= 12"
      }
    },
    "names": {
      "$n": {
        ".read": "auth != null",
        ".write": "auth != null && ((!data.exists() && newData.val() == auth.uid) || (data.val() == auth.uid && (!newData.exists() || newData.val() == auth.uid)))"
      }
    },
    "friends": {
      "$u": {
        ".read": "auth != null && auth.uid == $u",
        "$f": {
          ".write": "auth != null && (auth.uid == $u || (auth.uid == $f && (!newData.exists() || root.child('requests').child($f).child($u).exists())))"
        }
      }
    },
    "requests": {
      "$to": {
        ".read": "auth != null && auth.uid == $to",
        "$from": {
          ".write": "auth != null && ((auth.uid == $from && newData.hasChild('name') && root.child('users').child(auth.uid).exists()) || (auth.uid == $to && !newData.exists()))"
        }
      }
    }
  }
}
```

## Good to know
- Each browser gets its own anonymous account. If a player clears site data, they start over with a new name.
- Names are unique and not case sensitive.
- The free Firebase plan is plenty for a hobby game.
