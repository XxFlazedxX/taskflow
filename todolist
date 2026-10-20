from flask import Flask, render_template_string, request, redirect, url_for, session
import sqlite3
import os
from datetime import datetime

app = Flask(__name__)
app.secret_key = os.environ["SECRET_KEY"]  # Required for sessions
DATABASE = 'tasks.db'

ALL_ROLES = [
    "Product Manager", "Software Engineer", "Senior Software Engineer", "Frontend Developer", 
    "Backend Developer", "Full Stack Developer", "Data Scientist", "Data Analyst", 
    "Machine Learning Engineer", "UI/UX Designer", "Graphic Designer", "DevOps Engineer", 
    "Cloud Architect", "Cybersecurity Analyst", "QA Engineer", "Project Manager", 
    "Scrum Master", "Marketing Manager", "Sales Representative", "HR Specialist", 
    "Content Writer", "Accountant", "Student", "Freelancer", "CEO / Founder"
]

def init_db():
    with sqlite3.connect(DATABASE) as conn:
        cursor = conn.cursor()
        
        # Tasks Table
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS tasks (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER NOT NULL,
                title TEXT NOT NULL,
                due_datetime TEXT,
                priority TEXT,
                status TEXT DEFAULT 'doing'
            )
        ''')
        
        # Users Table
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT UNIQUE NOT NULL,
                email TEXT NOT NULL,
                role TEXT DEFAULT 'Product Manager'
            )
        ''')
        
        cursor.execute("PRAGMA table_info(tasks)")
        task_columns = [column[1] for column in cursor.fetchall()]
        
        if 'user_id' not in task_columns:
            print("Upgrading database: adding 'user_id' column to tasks...")
            cursor.execute("ALTER TABLE tasks ADD COLUMN user_id INTEGER DEFAULT 1")
        
        if 'due' in task_columns and 'due_datetime' not in task_columns:
            cursor.execute("ALTER TABLE tasks RENAME COLUMN due TO due_datetime")
        elif 'due_date' in task_columns and 'due_datetime' not in task_columns:
            cursor.execute("ALTER TABLE tasks RENAME COLUMN due_date TO due_datetime")
        
        conn.commit()

def get_db_connection():
    conn = sqlite3.connect(DATABASE)
    conn.row_factory = sqlite3.Row
    return conn

def format_date_for_display(date_str):
    if not date_str:
        return "No date set"
    try:
        dt = datetime.strptime(date_str, '%Y-%m-%dT%H:%M')
        return dt.strftime('%b %d, %Y at %I:%M %p')
    except Exception:
        return date_str

def get_avatar_gradient(role):
    """Returns a gradient based on the user's role for a unique look."""
    role_lower = role.lower()
    if 'engineer' in role_lower or 'developer' in role_lower or 'architect' in role_lower:
        return "linear-gradient(135deg, #3b82f6, #06b6d4)" # Blue/Cyan
    elif 'design' in role_lower or 'ux' in role_lower or 'ui' in role_lower:
        return "linear-gradient(135deg, #ec4899, #f43f5e)" # Pink/Red
    elif 'data' in role_lower or 'machine' in role_lower or 'analyst' in role_lower:
        return "linear-gradient(135deg, #8b5cf6, #d946ef)" # Purple/Magenta
    elif 'manager' in role_lower or 'scrum' in role_lower or 'ceo' in role_lower:
        return "linear-gradient(135deg, #f59e0b, #ef4444)" # Orange/Red
    elif 'marketing' in role_lower or 'sales' in role_lower or 'content' in role_lower:
        return "linear-gradient(135deg, #10b981, #3b82f6)" # Green/Blue
    else:
        return "linear-gradient(135deg, #6366f1, #8b5cf6)" # Default Indigo


LOGIN_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login - TaskFlow</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Inter', sans-serif; }
        body { display: flex; align-items: center; justify-content: center; min-height: 100vh; background: linear-gradient(135deg, #1e1b4b 0%, #312e81 100%); color: #111827; }
        .login-card { background: white; padding: 40px; border-radius: 16px; box-shadow: 0 20px 25px -5px rgba(0,0,0,0.2); width: 100%; max-width: 420px; }
        .login-card h2 { font-size: 28px; font-weight: 800; text-align: center; margin-bottom: 8px; background: linear-gradient(to right, #6366f1, #8b5cf6); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .login-card p.subtitle { text-align: center; color: #6b7280; font-size: 14px; margin-bottom: 30px; }
        .form-group { margin-bottom: 20px; }
        .form-group label { display: block; font-size: 13px; font-weight: 600; margin-bottom: 8px; color: #111827; }
        .form-group input, .form-group select { width: 100%; padding: 12px 16px; border: 1px solid #e5e7eb; border-radius: 8px; outline: none; font-size: 14px; transition: all 0.2s; background: #f9fafb; }
        .form-group input:focus, .form-group select:focus { border-color: #6366f1; box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.2); background: white; }
        .btn-login { width: 100%; padding: 14px; background: #6366f1; color: white; border: none; border-radius: 8px; font-weight: 700; font-size: 16px; cursor: pointer; transition: all 0.2s; box-shadow: 0 4px 12px rgba(99, 102, 241, 0.4); margin-top: 10px; }
        .btn-login:hover { background: #4f46e5; transform: translateY(-2px); }
    </style>
</head>
<body>
    <div class="login-card">
        <h2>TaskFlow</h2>
        <p class="subtitle">Welcome back! Please enter your details.</p>
        <form action="/login" method="POST">
            <div class="form-group">
                <label>Username</label>
                <input type="text" name="username" placeholder="e.g. johndoe" required>
            </div>
            <div class="form-group">
                <label>Email</label>
                <input type="email" name="email" placeholder="john@example.com" required>
            </div>
            <div class="form-group">
                <label>Your Role</label>
                <select name="role" required>
                    {% for role in roles %}
                    <option value="{{ role }}" {% if role == 'Software Engineer' %}selected{% endif %}>{{ role }}</option>
                    {% endfor %}
                </select>
            </div>
            <button type="submit" class="btn-login">Login / Create Account</button>
        </form>
    </div>
</body>
</html>
"""

DASHBOARD_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TaskFlow</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #6366f1; --primary-hover: #4f46e5; --bg-color: #f3f4f6;
            --sidebar-bg: #1e1b4b; --card-bg: #ffffff; --text-main: #111827;
            --text-muted: #6b7280; --border: #e5e7eb;
            --high: #ef4444; --medium: #f59e0b; --low: #10b981;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Inter', sans-serif; }
        body { display: flex; height: 100vh; background-color: var(--bg-color); color: var(--text-main); }
        
        /* Sidebar */
        .sidebar { width: 260px; background-color: var(--sidebar-bg); padding: 30px 20px; color: white; display: flex; flex-direction: column; }
        .sidebar h2 { font-size: 24px; font-weight: 800; margin-bottom: 40px; letter-spacing: 1px; background: linear-gradient(to right, #a5b4fc, #818cf8); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .sidebar ul { list-style: none; display: flex; flex-direction: column; gap: 8px; }
        .sidebar a { text-decoration: none; color: #a5b4fc; padding: 12px 16px; border-radius: 8px; display: block; font-weight: 500; transition: all 0.2s; }
        .sidebar a:hover { background-color: rgba(255,255,255,0.1); color: white; transform: translateX(4px); }
        .sidebar .active a { background-color: var(--primary); color: white; box-shadow: 0 4px 12px rgba(99, 102, 241, 0.4); }
        
        /* Main Content */
        .main-content { flex: 1; display: flex; flex-direction: column; overflow: hidden; }
        .topbar { height: 70px; background-color: var(--card-bg); border-bottom: 1px solid var(--border); display: flex; align-items: center; justify-content: space-between; padding: 0 30px; z-index: 10; }
        .search-bar { padding: 10px 16px; border: 1px solid var(--border); border-radius: 20px; width: 300px; outline: none; transition: border 0.2s; background: var(--bg-color); }
        .search-bar:focus { border-color: var(--primary); background: white; box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.1); }
        
        /* User Profile */
        .user-menu-container { position: relative; }
        .user-btn { display: flex; align-items: center; gap: 12px; background: var(--bg-color); padding: 6px 16px 6px 6px; border-radius: 30px; cursor: pointer; border: 1px solid var(--border); transition: all 0.2s; }
        .user-btn:hover { border-color: var(--primary); box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
        .avatar { width: 36px; height: 36px; border-radius: 50%; background: {{ avatar_gradient }}; color: white; display: flex; align-items: center; justify-content: center; font-weight: 700; font-size: 14px; }
        .user-info { display: flex; flex-direction: column; }
        .user-info .name { font-size: 14px; font-weight: 600; color: var(--text-main); }
        .user-info .role { font-size: 11px; color: var(--text-muted); }
        .dropdown-menu { position: absolute; top: 110%; right: 0; background: white; border-radius: 12px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); border: 1px solid var(--border); width: 200px; padding: 8px; display: none; flex-direction: column; }
        .dropdown-menu.show { display: flex; }
        .dropdown-menu a { padding: 10px 12px; text-decoration: none; color: var(--text-main); border-radius: 8px; font-size: 14px; display: flex; align-items: center; gap: 10px; }
        .dropdown-menu a:hover { background: var(--bg-color); color: var(--primary); }
        .dropdown-menu hr { border: none; border-top: 1px solid var(--border); margin: 4px 0; }
        .dropdown-menu .logout { color: var(--high); }

        /* Dashboard */
        .dashboard { padding: 30px; overflow-y: auto; flex: 1; }
        .dashboard h1 { font-size: 28px; margin-bottom: 24px; font-weight: 800; }
        .dashboard h1 span { color: var(--primary); }
        
        /* Stats */
        .stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; margin-bottom: 30px; }
        .stat-card { background: var(--card-bg); padding: 24px; border-radius: 12px; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05); border: 1px solid var(--border); display: flex; flex-direction: column; gap: 8px; position: relative; overflow: hidden; }
        .stat-card::before { content: ''; position: absolute; left: 0; top: 0; bottom: 0; width: 4px; }
        .stat-card:nth-child(1)::before { background: var(--primary); }
        .stat-card:nth-child(2)::before { background: var(--medium); }
        .stat-card:nth-child(3)::before { background: var(--low); }
        .stat-card span:first-child { font-size: 13px; color: var(--text-muted); font-weight: 600; text-transform: uppercase; letter-spacing: 0.5px; }
        .stat-card span:last-child { font-size: 32px; font-weight: 800; color: var(--text-main); }
        
        /* Task Section */
        .task-section { background: var(--card-bg); padding: 30px; border-radius: 12px; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05); border: 1px solid var(--border); }
        .task-header { margin-bottom: 24px; padding-bottom: 16px; border-bottom: 1px solid var(--border); display: flex; justify-content: space-between; align-items: center; }
        .task-header h2 { font-size: 20px; font-weight: 700; }
        
        /* Add Form */
        .add-task-form { display: flex; gap: 12px; margin-bottom: 30px; flex-wrap: wrap; background: var(--bg-color); padding: 20px; border-radius: 12px; }
        .add-task-form input, .add-task-form select { padding: 12px 16px; border: 1px solid var(--border); border-radius: 8px; outline: none; font-size: 14px; transition: all 0.2s; background: white; }
        .add-task-form input:focus, .add-task-form select:focus { border-color: var(--primary); box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.2); }
        .add-task-form input[type="text"] { flex: 1; min-width: 200px; }
        .add-task-form button { padding: 12px 24px; background: var(--primary); color: white; border: none; border-radius: 8px; font-weight: 600; cursor: pointer; transition: all 0.2s; box-shadow: 0 4px 12px rgba(99, 102, 241, 0.3); }
        .add-task-form button:hover { background: var(--primary-hover); transform: translateY(-2px); }

        /* Task Item */
        .task-item { display: flex; justify-content: space-between; align-items: center; padding: 20px; border-bottom: 1px solid var(--border); transition: background 0.2s; }
        .task-item:hover { background: #f8fafc; }
        .task-item:last-child { border-bottom: none; }
        .task-info h3 { font-size: 16px; margin-bottom: 6px; font-weight: 600; color: var(--text-main); }
        .task-info p { font-size: 14px; color: var(--text-muted); display: flex; align-items: center; gap: 6px; }
        .task-actions { display: flex; align-items: center; gap: 16px; }
        
        /* Badges */
        .status-form select { padding: 6px 12px; border-radius: 20px; border: none; font-weight: 600; font-size: 12px; cursor: pointer; outline: none; appearance: none; text-align: center; }
        .status-doing { background-color: #e0e7ff; color: #4338ca; }
        .status-completed { background-color: #d1fae5; color: #047857; }
        .status-upcoming { background-color: #fef3c7; color: #b45309; }
        .priority-badge { font-size: 12px; font-weight: 600; padding: 6px 12px; border-radius: 20px; }
        .priority-High { background-color: #fee2e2; color: var(--high); }
        .priority-Medium { background-color: #fef3c7; color: var(--medium); }
        .priority-Low { background-color: #d1fae5; color: var(--low); }
        .btn-remove { color: #ef4444; text-decoration: none; font-size: 14px; font-weight: 600; padding: 6px 12px; border-radius: 6px; transition: background 0.2s; }
        .btn-remove:hover { background: #fee2e2; }

        /* Modals */
        .modal-overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(17, 24, 39, 0.6); backdrop-filter: blur(4px); display: none; align-items: center; justify-content: center; z-index: 1000; }
        .modal-overlay.show { display: flex; }
        .modal { background: white; border-radius: 16px; padding: 30px; width: 90%; max-width: 400px; box-shadow: 0 20px 25px -5px rgba(0,0,0,0.1); animation: slideUp 0.3s ease; }
        @keyframes slideUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
        .modal h3 { font-size: 20px; margin-bottom: 20px; font-weight: 700; }
        .form-group { margin-bottom: 16px; }
        .form-group label { display: block; font-size: 13px; font-weight: 600; margin-bottom: 6px; color: var(--text-main); }
        .form-group input, .form-group select { width: 100%; padding: 10px 14px; border: 1px solid var(--border); border-radius: 8px; outline: none; font-size: 14px; }
        .form-group input:focus, .form-group select:focus { border-color: var(--primary); box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.2); }
        .modal-actions { display: flex; justify-content: flex-end; gap: 10px; margin-top: 24px; }
        .btn { padding: 10px 20px; border-radius: 8px; font-weight: 600; font-size: 14px; cursor: pointer; border: none; transition: all 0.2s; }
        .btn-secondary { background: var(--bg-color); color: var(--text-main); }
        .btn-secondary:hover { background: #e5e7eb; }
        .btn-primary { background: var(--primary); color: white; }
        .btn-primary:hover { background: var(--primary-hover); }
    </style>
</head>
<body>
    <!-- Sidebar -->
    <div class="sidebar">
        <h2>TaskFlow</h2>
        <ul>
            <li class="{% if current_tab == 'dashboard' %}active{% endif %}"><a href="/?tab=dashboard">Dashboard</a></li>
            <li class="{% if current_tab == 'all' %}active{% endif %}"><a href="/?tab=all">All Tasks</a></li>
            <li class="{% if current_tab == 'upcoming' %}active{% endif %}"><a href="/?tab=upcoming">Upcoming</a></li>
            <li class="{% if current_tab == 'completed' %}active{% endif %}"><a href="/?tab=completed">Completed</a></li>
            <li class="{% if current_tab == 'doing' %}active{% endif %}"><a href="/?tab=doing">Doing</a></li>
        </ul>
    </div>

    <!-- Main Content -->
    <div class="main-content">
        <!-- Topbar -->
        <div class="topbar">
            <input type="text" class="search-bar" placeholder="Search tasks...">
            
            <!-- User Profile Area -->
            <div class="user-menu-container">
                <div class="user-btn" onclick="toggleDropdown()">
                    <div class="avatar" style="background: {{ avatar_gradient }}">{{ user.initials }}</div>
                    <div class="user-info">
                        <span class="name">{{ user.username }}</span>
                        <span class="role">{{ user.role }}</span>
                    </div>
                    <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="margin-left:4px; color:#6b7280;"><polyline points="6 9 12 15 18 9"></polyline></svg>
                </div>
                
                <!-- Dropdown Menu -->
                <div class="dropdown-menu" id="userDropdown">
                    <a href="javascript:void(0)" onclick="openModal('profileModal')">
                        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path><circle cx="12" cy="7" r="4"></circle></svg>
                        My Profile
                    </a>
                    <a href="javascript:void(0)" onclick="openModal('settingsModal')">
                        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="3"></circle><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path></svg>
                        Settings
                    </a>
                    <hr>
                    <a href="/logout" class="logout">
                        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path><polyline points="16 17 21 12 16 7"></polyline><line x1="21" y1="12" x2="9" y2="12"></line></svg>
                        Log out
                    </a>
                </div>
            </div>
        </div>

        <!-- Dashboard -->
        <div class="dashboard">
            <h1>Welcome back, <span>{{ user.username.split(' ')[0] }}</span> 👋</h1>
            
            <!-- Stats -->
            <div class="stats">
                <div class="stat-card">
                    <span>Total Tasks</span>
                    <span>{{ total }}</span>
                </div>
                <div class="stat-card">
                    <span>Doing</span>
                    <span>{{ doing_count }}</span>
                </div>
                <div class="stat-card">
                    <span>Completed</span>
                    <span>{{ completed_count }}</span>
                </div>
            </div>

            <!-- Task Area -->
            <div class="task-section">
                <div class="task-header">
                    <h2>{{ tab_title }}</h2>
                </div>

                <!-- Add Task Form -->
                <form class="add-task-form" action="/add" method="POST">
                    <input type="text" name="title" placeholder="What needs to be done?" required>
                    <input type="datetime-local" name="due_datetime" title="Due Date & Time" required>
                    <select name="status">
                        <option value="doing" selected>Doing</option>
                        <option value="completed">Completed</option>
                        <option value="upcoming">Upcoming</option>
                    </select>
                    <select name="priority">
                        <option value="Low">Low Priority</option>
                        <option value="Medium" selected>Medium Priority</option>
                        <option value="High">High Priority</option>
                    </select>
                    <button type="submit">+ Add Task</button>
                </form>

                <!-- Task List -->
                {% for task in tasks %}
                <div class="task-item">
                    <div class="task-info">
                        <h3>{{ task['title'] }}</h3>
                        <p>📅 {{ task['formatted_date'] }}</p>
                    </div>
                    <div class="task-actions">
                        <form class="status-form" action="/update_status/{{ task['id'] }}" method="POST">
                            <select name="status" class="status-{{ task['status'] }}" onchange="this.form.submit()" title="Change Status">
                                <option value="doing" {% if task['status'] == 'doing' %}selected{% endif %}>Doing</option>
                                <option value="completed" {% if task['status'] == 'completed' %}selected{% endif %}>Completed</option>
                                <option value="upcoming" {% if task['status'] == 'upcoming' %}selected{% endif %}>Upcoming</option>
                            </select>
                        </form>
                        <span class="priority-badge priority-{{ task['priority'] }}">{{ task['priority'] }}</span>
                        <a href="/remove/{{ task['id'] }}" class="btn-remove">Remove</a>
                    </div>
                </div>
                {% else %}
                <div style="text-align: center; padding: 40px; color: var(--text-muted);">
                    <p style="font-size: 18px; margin-bottom: 8px;">No tasks found here.</p>
                    <p style="font-size: 14px;">Add a task using the form above!</p>
                </div>
                {% endfor %}
            </div>
        </div>
    </div>

    <!-- Profile Modal -->
    <div class="modal-overlay" id="profileModal">
        <div class="modal">
            <h3>My Profile</h3>
            <div style="display: flex; align-items: center; gap: 16px; margin-bottom: 20px;">
                <div class="avatar" style="width: 60px; height: 60px; font-size: 24px; background: {{ avatar_gradient }}">{{ user.initials }}</div>
                <div>
                    <h4 style="font-size: 18px; font-weight: 700; margin-bottom: 4px;">{{ user.username }}</h4>
                    <p style="color: var(--text-muted); font-size: 14px;">{{ user.role }}</p>
                </div>
            </div>
            <div style="background: var(--bg-color); padding: 16px; border-radius: 8px;">
                <p style="font-size: 13px; color: var(--text-muted); margin-bottom: 8px;">Email Address</p>
                <p style="font-size: 14px; font-weight: 600;">{{ user.email }}</p>
            </div>
            <div class="modal-actions">
                <button class="btn btn-primary" onclick="closeModal('profileModal')">Close</button>
            </div>
        </div>
    </div>

    <!-- Settings Modal -->
    <div class="modal-overlay" id="settingsModal">
        <div class="modal">
            <h3>Account Settings</h3>
            <form action="/update_user" method="POST">
                <div class="form-group">
                    <label>Username</label>
                    <input type="text" name="username" value="{{ user.username }}" required>
                </div>
                <div class="form-group">
                    <label>Email</label>
                    <input type="email" name="email" value="{{ user.email }}" required>
                </div>
                <div class="form-group">
                    <label>Role</label>
                    <select name="role" required>
                        {% for role in roles %}
                        <option value="{{ role }}" {% if user.role == role %}selected{% endif %}>{{ role }}</option>
                        {% endfor %}
                    </select>
                </div>
                <div class="modal-actions" style="justify-content: space-between;">
                    <a href="/logout" class="btn" style="background:#fee2e2; color:#ef4444; text-decoration:none;">Log Out & Switch</a>
                    <div style="display:flex; gap:10px;">
                        <button type="button" class="btn btn-secondary" onclick="closeModal('settingsModal')">Cancel</button>
                        <button type="submit" class="btn btn-primary">Save</button>
                    </div>
                </div>
            </form>
        </div>
    </div>

    <script>
        function toggleDropdown() { document.getElementById('userDropdown').classList.toggle('show'); }
        window.onclick = function(event) {
            if (!event.target.closest('.user-menu-container')) {
                document.getElementById('userDropdown').classList.remove('show');
            }
        }
        function openModal(id) { document.getElementById(id).classList.add('show'); document.getElementById('userDropdown').classList.remove('show'); }
        function closeModal(id) { document.getElementById(id).classList.remove('show'); }
    </script>
</body>
</html>
"""


@app.route('/')
def index():
    if 'user_id' not in session:
        return redirect(url_for('login'))
    
    tab = request.args.get('tab', 'dashboard')
    conn = get_db_connection()
    
    # Fetch current user
    user = conn.execute('SELECT * FROM users WHERE id = ?', (session['user_id'],)).fetchone()
    if not user:
        session.clear()
        return redirect(url_for('login'))
    user = dict(user)
    
    # Get tasks ONLY for this user
    all_tasks = conn.execute('SELECT * FROM tasks WHERE user_id = ?', (user['id'],)).fetchall()
    total = len(all_tasks)
    doing_count = len([t for t in all_tasks if t['status'] == 'doing'])
    completed_count = len([t for t in all_tasks if t['status'] == 'completed'])
    
    if tab == 'doing':
        tasks = conn.execute('SELECT * FROM tasks WHERE user_id = ? AND status = "doing" ORDER BY due_datetime ASC', (user['id'],)).fetchall()
        tab_title = "Doing Tasks"
    elif tab == 'completed':
        tasks = conn.execute('SELECT * FROM tasks WHERE user_id = ? AND status = "completed" ORDER BY due_datetime DESC', (user['id'],)).fetchall()
        tab_title = "Completed Tasks"
    elif tab == 'upcoming':
        tasks = conn.execute('SELECT * FROM tasks WHERE user_id = ? AND status = "upcoming" ORDER BY due_datetime ASC', (user['id'],)).fetchall()
        tab_title = "Upcoming Tasks"
    elif tab == 'all':
        tasks = conn.execute('SELECT * FROM tasks WHERE user_id = ? ORDER BY due_datetime ASC', (user['id'],)).fetchall()
        tab_title = "All Tasks"
    else:
        tasks = conn.execute('SELECT * FROM tasks WHERE user_id = ? ORDER BY due_datetime ASC', (user['id'],)).fetchall()
        tab_title = "Dashboard Overview"
        
    conn.close()
    
    formatted_tasks = []
    for t in tasks:
        task_dict = dict(t)
        task_dict['formatted_date'] = format_date_for_display(t['due_datetime'])
        formatted_tasks.append(task_dict)
        
    name_parts = user['username'].split()
    initials = "".join([part[0].upper() for part in name_parts if part])[:2]
    user['initials'] = initials if initials else "U"
    
    return render_template_string(DASHBOARD_TEMPLATE, 
                                  tasks=formatted_tasks, 
                                  total=total, 
                                  doing_count=doing_count, 
                                  completed_count=completed_count,
                                  current_tab=tab,
                                  tab_title=tab_title,
                                  user=user,
                                  roles=ALL_ROLES,
                                  avatar_gradient=get_avatar_gradient(user['role']))

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form.get('username').strip()
        email = request.form.get('email').strip()
        role = request.form.get('role')
        
        conn = get_db_connection()
        user = conn.execute('SELECT * FROM users WHERE username = ?', (username,)).fetchone()
        
        if user:
            conn.execute('UPDATE users SET email = ?, role = ? WHERE id = ?', (email, role, user['id']))
            user_id = user['id']
        else:
            cursor = conn.cursor()
            cursor.execute('INSERT INTO users (username, email, role) VALUES (?, ?, ?)', (username, email, role))
            user_id = cursor.lastrowid
        
        conn.commit()
        conn.close()
        
        session['user_id'] = user_id
        return redirect(url_for('index'))
        
    return render_template_string(LOGIN_TEMPLATE, roles=ALL_ROLES)

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('login'))

@app.route('/add', methods=['POST'])
def add_task():
    if 'user_id' not in session: return redirect(url_for('login'))
    
    title = request.form.get('title')
    due_datetime = request.form.get('due_datetime')
    priority = request.form.get('priority')
    status = request.form.get('status')
    
    if title:
        conn = get_db_connection()
        conn.execute('INSERT INTO tasks (user_id, title, due_datetime, priority, status) VALUES (?, ?, ?, ?, ?)',
                     (session['user_id'], title, due_datetime, priority, status))
        conn.commit()
        conn.close()
        
    return redirect(request.referrer or url_for('index'))

@app.route('/update_status/<int:id>', methods=['POST'])
def update_status(id):
    if 'user_id' not in session: return redirect(url_for('login'))
    new_status = request.form.get('status')
    if new_status:
        conn = get_db_connection()
        conn.execute('UPDATE tasks SET status = ? WHERE id = ? AND user_id = ?', (new_status, id, session['user_id']))
        conn.commit()
        conn.close()
    return redirect(request.referrer or url_for('index'))

@app.route('/remove/<int:id>')
def remove_task(id):
    if 'user_id' not in session: return redirect(url_for('login'))
    conn = get_db_connection()
    conn.execute('DELETE FROM tasks WHERE id = ? AND user_id = ?', (id, session['user_id']))
    conn.commit()
    conn.close()
    return redirect(request.referrer or url_for('index'))

@app.route('/update_user', methods=['POST'])
def update_user():
    if 'user_id' not in session: return redirect(url_for('login'))
    
    username = request.form.get('username').strip()
    email = request.form.get('email').strip()
    role = request.form.get('role')
    
    if username and email and role:
        conn = get_db_connection()
        existing = conn.execute('SELECT id FROM users WHERE username = ? AND id != ?', (username, session['user_id'])).fetchone()
        if not existing:
            conn.execute('UPDATE users SET username = ?, email = ?, role = ? WHERE id = ?', (username, email, role, session['user_id']))
            conn.commit()
        conn.close()
        
    return redirect(request.referrer or url_for('index'))

init_db()

if __name__ == '__main__':
    app.run(debug=True)
