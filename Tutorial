# Session Storage Issue Fix Project

## Overview

This project demonstrates fixing Django session storage issues and ensures session data can be set, retrieved, and cleared properly. It also addresses the CORS issue that occurs when opening HTML files directly from the filesystem.

### Key Points

* Django sessions are stored server-side, and they are only saved when modified.
* Browsers cannot directly see full session data; only the session cookie (`sessionid`) is sent.
* To avoid CORS issues, always serve pages through Django development server rather than `file:///` protocol.

---

## 1. Django Views

```python
# views.py
from django.shortcuts import render
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
import json


def session_test_page(request):
    """Serve the session test page"""
    return render(request, 'session_test.html')


def session_test(request):
    """API endpoint to get session data"""
    session_data = {
        'currency': request.session.get('currency', 'Not set'),
        'language': request.session.get('language', 'Not set'),
        'timezone': request.session.get('timezone', 'Not set'),
        'session_key': request.session.session_key,
    }
    return JsonResponse(session_data)


@csrf_exempt
def set_session_manual(request):
    """API endpoint to manually set session values"""
    if request.method == 'POST':
        try:
            data = json.loads(request.body)
            request.session['currency'] = data.get('currency', 'USD')
            request.session['language'] = data.get('language', 'en')
            request.session['timezone'] = data.get('timezone', 'UTC')
            request.session.modified = True
            return JsonResponse({'status': 'success', 'message': 'Session values set manually'})
        except Exception as e:
            return JsonResponse({'status': 'error', 'message': str(e)})

    return JsonResponse({'status': 'error', 'message': 'Only POST requests allowed'})


def clear_session(request):
    """API endpoint to clear session"""
    request.session.flush()
    return JsonResponse({'status': 'success', 'message': 'Session cleared'})
```

---

## 2. URLs

```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('session-test-page/', views.session_test_page, name='session_test_page'),
    path('session-test/', views.session_test, name='session_test'),
    path('set-session/', views.set_session_manual, name='set_session_manual'),
    path('clear-session/', views.clear_session, name='clear_session'),
]
```

---

## 3. Template: `session_test.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>Session Test</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        .section { margin-bottom: 20px; padding: 15px; border: 1px solid #ddd; border-radius: 5px; }
        button { padding: 8px 15px; margin: 5px; background: #4CAF50; color: white; border: none; border-radius: 4px; cursor: pointer; }
        button:hover { background: #45a049; }
        pre { background: #f4f4f4; padding: 10px; border-radius: 4px; }
        select { padding: 8px; margin: 5px; }
    </style>
</head>
<body>
    <h1>Session Test Page</h1>
    <p>Testing session cookie functionality with SameSite=None</p>
    
    <div class="section">
        <h2>Current Session Values</h2>
        <pre id="session-data">Loading...</pre>
        <button onclick="loadSession()">Refresh Session Data</button>
        <button onclick="clearSession()">Clear Session</button>
    </div>
    
    <div class="section">
        <h2>Set Session Manually</h2>
        <select id="currency">
            <option value="USD">USD</option>
            <option value="BDT">BDT</option>
            <option value="INR">INR</option>
        </select>
        <select id="language">
            <option value="en">English</option>
            <option value="bn">Bengali</option>
            <option value="hi">Hindi</option>
        </select>
        <button onclick="setSession()">Set Session Values</button>
    </div>

    <div class="section">
        <h2>Debug Info</h2>
        <p id="debug-info"></p>
    </div>

    <script>
        function makeRequest(method, url, data, callback) {
            const xhr = new XMLHttpRequest();
            xhr.open(method, url, true);
            xhr.setRequestHeader('Content-Type', 'application/json');
            xhr.withCredentials = true;
            xhr.onreadystatechange = function() {
                if (xhr.readyState === 4) {
                    if (xhr.status === 200) {
                        callback(JSON.parse(xhr.responseText));
                    } else {
                        document.getElementById('debug-info').textContent = 'Error: ' + xhr.status + ' - ' + xhr.statusText;
                    }
                }
            };
            if (data) xhr.send(JSON.stringify(data));
            else xhr.send();
        }
        
        function loadSession() {
            makeRequest('GET', '/session-test/', null, function(data) {
                document.getElementById('session-data').textContent = JSON.stringify(data, null, 2);
                document.getElementById('debug-info').textContent = 'Session loaded successfully. Cookie should be set with SameSite=None; Secure';
            });
        }
        
        function setSession() {
            const currency = document.getElementById('currency').value;
            const language = document.getElementById('language').value;
            makeRequest('POST', '/set-session/', {currency: currency, language: language, timezone: 'UTC'}, function(data) {
                alert('Session set: ' + JSON.stringify(data));
                loadSession();
            });
        }
        
        function clearSession() {
            makeRequest('GET', '/clear-session/', null, function(data) {
                alert('Session cleared: ' + JSON.stringify(data));
                loadSession();
            });
        }

        document.addEventListener('DOMContentLoaded', function() {
            loadSession();
            document.getElementById('debug-info').textContent = 'Page loaded. Check browser dev tools to see if session cookie is set properly.';
        });
    </script>
</body>
</html>
```

---

## 4. Development Server

Run the Django development server:

```
python manage.py runserver
```

Access the page via:

```
http://127.0.0.1:8000/session-test-page/
```

---

## 5. Django Settings

```python
# settings.py
SESSION_COOKIE_SAMESITE = 'None'
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SAMESITE = 'None'
CSRF_COOKIE_SECURE = True

# Optional for development
if DEBUG:
    import socket
    hostname, _, ips = socket.gethostbyname_ex(socket.gethostname())
    INTERNAL_IPS = [ip[: ip.rfind('.')]+'.1' for ip in ips] + ['127.0.0.1', '10.0.2.2']
```

---

## 6. Notes

* Always serve pages via Django server to avoid CORS issues.
* Use native JavaScript for AJAX requests; `withCredentials = true` ensures cookies are sent.
* Session data is stored server-side; only `sessionid` cookie is visible in browser developer tools.
* Use `request.session.modified = True` to ensure changes are saved.
