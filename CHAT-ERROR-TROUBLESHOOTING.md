# 🔧 Chat Error Troubleshooting Guide

## Problem: Getting "[object Object]" Error

When you ask questions in the chat (like "what products should I use for my oily skin?"), you're seeing:
```
Sorry, I encountered an error: [object Object]. Please try again.
```

## ✅ What I Fixed

I've improved the error handling in three ways:

### 1. **Better Error Message Extraction** (script.js line ~653)
The app now properly extracts error messages from various error formats instead of showing `[object Object]`.

### 2. **Network Error Handling** (script.js line ~605)
Added try-catch around the fetch to catch network errors and show a clear message.

### 3. **JSON Parse Error Handling** (script.js line ~630)
Added error handling for invalid JSON responses from the server.

## 🧪 How to Diagnose the Issue

### Step 1: Open the Test Page
1. Open `test-chat.html` in your browser
2. Click **"Test Oily Skin Question"** button
3. Check the results

### Step 2: Check Browser Console
1. Press **F12** to open Developer Tools
2. Go to the **Console** tab
3. Try asking the question again in the main app
4. Look for error messages in red

### Step 3: Check Network Tab
1. In Developer Tools, go to **Network** tab
2. Try asking the question again
3. Look for the request to `loreal-routine-builder.esjohn15.workers.dev`
4. Click on it and check:
   - **Status Code** (should be 200)
   - **Response** tab (see what the server sent back)
   - **Preview** tab (formatted JSON view)

## 🔍 Common Issues & Solutions

### Issue 1: Network Error (Can't Connect)
**Error Message:** `Network error: Unable to connect to the API`

**Possible Causes:**
- No internet connection
- Cloudflare Worker is down
- CORS issues

**Solutions:**
1. Check your internet connection
2. Try the curl test:
```bash
curl -X POST https://loreal-routine-builder.esjohn15.workers.dev/ \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {"role": "user", "content": "Hello"}
    ]
  }'
```
3. If curl works but browser doesn't, check for CORS errors in console

### Issue 2: Server Error (HTTP 500, 502, etc.)
**Error Message:** `Server error (500): ...`

**Possible Causes:**
- Cloudflare Worker error
- OpenAI API key issues
- Rate limiting

**Solutions:**
1. Check Cloudflare Worker logs:
```bash
cd "/Users/lizziejohnson/Desktop/GIT337-Field and Feist assingment/final-routine-builder"
wrangler tail
```
2. Verify API secrets are set:
```bash
wrangler secret list
```
3. Should see:
   - `OPENAI_API_KEY`
   - `MISTRAL_API_KEY`

### Issue 3: Invalid Response Format
**Error Message:** `Invalid response from API`

**Possible Causes:**
- API returned non-JSON response
- API changed response format
- Cloudflare Worker bug

**Solutions:**
1. Check the Network tab response
2. Look for HTML error pages (500 errors)
3. Test with `test-chat.html` to see raw response

### Issue 4: API Key Issues
**Error Message:** May mention authentication or unauthorized

**Possible Causes:**
- OpenAI API key expired or invalid
- Insufficient credits on OpenAI account
- Wrong API key format

**Solutions:**
1. Verify API key in Cloudflare dashboard
2. Check OpenAI account status: https://platform.openai.com/account/usage
3. Check for rate limiting errors

### Issue 5: CORS Errors
**Error Message:** Console shows CORS policy errors

**Possible Causes:**
- Cloudflare Worker not sending correct CORS headers
- Browser security settings

**Solutions:**
1. Check worker.js has CORS headers:
```javascript
headers: {
  'Access-Control-Allow-Origin': '*',
  'Content-Type': 'application/json'
}
```
2. Redeploy worker if headers are missing:
```bash
wrangler deploy
```

## 🎯 Step-by-Step Debugging Process

### 1. Test the API Directly (Outside Browser)
```bash
curl -X POST https://loreal-routine-builder.esjohn15.workers.dev/ \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "What products should I use for oily skin?"}
    ],
    "temperature": 0.7,
    "max_tokens": 500
  }'
```

**If this works:** Issue is in the browser/frontend code  
**If this fails:** Issue is in the Cloudflare Worker or API keys

### 2. Test in Browser with test-chat.html
1. Open `test-chat.html`
2. Open browser console (F12)
3. Click "Test Oily Skin Question"
4. Check both the page output AND console logs

**If this works:** Issue is specific to the main app  
**If this fails:** Issue is browser-related (CORS, network, etc.)

### 3. Test in Main App with Console Open
1. Open `index.html`
2. Open browser console (F12)
3. Ask the question in the chat
4. Look for detailed error logs

**New logs you'll see:**
- `Sending X messages to AI`
- `Response status: 200`
- `Response data: {...}`
- Detailed error messages if something fails

### 4. Check Cloudflare Worker Logs
```bash
cd "/Users/lizziejohnson/Desktop/GIT337-Field and Feist assingment/final-routine-builder"
wrangler tail
```

Then try the chat again. You'll see real-time logs from the worker.

## 📊 What the Improved Error Messages Show

### Before (Unhelpful):
```
Sorry, I encountered an error: [object Object]. Please try again.
```

### After (Helpful):
```
Sorry, I encountered an error: Network error: Unable to connect to the API. Please check your internet connection.
```

OR

```
Sorry, I encountered an error: Server error (500): OpenAI API error: Insufficient credits.
```

OR

```
Sorry, I encountered an error: Invalid JSON response from server.
```

## 🔬 Advanced Debugging

### Enable Verbose Logging
Open `script.js` and the console logs will now show:
- Request payload being sent
- Response status code
- Full response data
- Network errors with details
- JSON parse errors

### Check Conversation History
The app saves conversation history in localStorage. To view:
```javascript
// In browser console:
JSON.parse(localStorage.getItem('loreal_conversation_history'))
```

### Clear Conversation History
If the conversation history is causing issues:
```javascript
// In browser console:
localStorage.removeItem('loreal_conversation_history')
```

Then refresh and try again.

## 🚀 Quick Fixes

### Fix 1: Clear Browser Cache
1. Press **Cmd+Shift+R** (Mac) or **Ctrl+Shift+R** (Windows)
2. This hard-refreshes and clears cached JavaScript

### Fix 2: Try Incognito Mode
1. Open browser in incognito/private mode
2. Navigate to your app
3. Test the chat
4. If it works, the issue is cached data or extensions

### Fix 3: Check Browser Console for Blocked Resources
1. Open Console (F12)
2. Look for red errors about blocked resources
3. Check if any browser extensions are blocking the API

### Fix 4: Verify Worker is Deployed
```bash
cd "/Users/lizziejohnson/Desktop/GIT337-Field and Feist assingment/final-routine-builder"
wrangler deployments list
```

Should show recent deployments.

## 📱 Mobile Testing

If testing on mobile:
1. Use **Remote Debugging** (Chrome DevTools)
2. Or use **Safari Web Inspector** (iOS)
3. Check for mobile-specific errors (CORS, CSP, etc.)

## ✅ Success Checklist

When everything works, you should see:

**In Browser Console:**
- ✅ `Sending 0 previous exchanges for context`
- ✅ `Response status: 200` (or similar)
- ✅ `Conversation history now has 1 exchanges`
- ✅ No red errors

**In Network Tab:**
- ✅ Request to worker URL shows status 200
- ✅ Response contains `choices` array
- ✅ Response time under 10 seconds

**In Chat Window:**
- ✅ AI response appears
- ✅ No error messages
- ✅ Follow-up questions work

## 🎓 Understanding the Error Flow

```
User asks question
    ↓
sendToOpenAI() called
    ↓
Try to fetch from Worker
    ↓
    ├─ Network Error? → "Network error: Unable to connect"
    ├─ HTTP Error (4xx/5xx)? → "Server error (500): ..."
    ├─ Invalid JSON? → "Invalid JSON response from server"
    └─ Success? → Parse response
        ↓
        ├─ No choices? → Extract error message from data.error
        └─ Success? → Show AI response
```

## 📞 Need More Help?

If you're still seeing errors:

1. **Run all tests** in `test-chat.html`
2. **Copy the console output** (all red errors)
3. **Copy the Network tab** response
4. **Check worker logs** with `wrangler tail`
5. Share these details for debugging

## 🎉 Expected Behavior

When working correctly:
1. You ask: "What products should I use for oily skin?"
2. Loading indicator appears
3. Within 3-7 seconds, AI responds with detailed recommendations
4. Response includes product suggestions for oily skin
5. You can ask follow-up questions

---

**Files Modified for Better Error Handling:**
- ✅ `script.js` - Lines ~605-680 (improved error handling)
- ✅ `script.js` - Line ~760 (chat error display)
- ✅ `test-chat.html` - NEW diagnostic tool

**Next Steps:**
1. Open `test-chat.html` in your browser
2. Run all three tests
3. Check if they pass
4. If they pass, try the main app again
5. If main app still fails, check browser console for specific errors

---

*Last Updated: November 15, 2025*  
*Error Handling v2.0* 🛠️
