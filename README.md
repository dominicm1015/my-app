<!doctype html>
<html lang="en" class="h-full">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Post-it Social</title>
  <script src="https://cdn.jsdelivr.net/npm/lucide@0.263.0/dist/umd/lucide.min.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    html, body { height: 100%; font-family: 'Outfit', sans-serif; }
    body { background: #0a0a0f; color: #f0f0f0; overflow: hidden; }
    .app-shell { height: 100%; width: 100%; display: flex; flex-direction: column; overflow: hidden; }
    .screen { display: none; flex-direction: column; height: 100%; width: 100%; overflow: hidden; }
    .screen.active { display: flex; }
    .gradient-bg { background: linear-gradient(135deg, #0a0a0f 0%, #1a1028 50%, #0a0a0f 100%); }
    .card-glass { background: rgba(255,255,255,0.06); border: 1px solid rgba(255,255,255,0.1); backdrop-filter: blur(12px); border-radius: 16px; }
    .btn-primary { background: linear-gradient(135deg, #a855f7, #6d28d9); color: white; border: none; border-radius: 12px; padding: 14px 28px; font-family: 'Outfit', sans-serif; font-weight: 600; font-size: 16px; cursor: pointer; transition: all 0.2s; width: 100%; }
    .btn-primary:hover { transform: translateY(-1px); box-shadow: 0 8px 24px rgba(168,85,247,0.3); }
    .btn-secondary { background: rgba(255,255,255,0.08); color: #ccc; border: 1px solid rgba(255,255,255,0.12); border-radius: 12px; padding: 12px 24px; font-family: 'Outfit', sans-serif; font-weight: 500; font-size: 14px; cursor: pointer; transition: all 0.2s; }
    .btn-secondary:hover { background: rgba(255,255,255,0.14); }
    .input-field { width: 100%; padding: 14px 16px; background: rgba(255,255,255,0.06); border: 1px solid rgba(255,255,255,0.12); border-radius: 12px; color: #f0f0f0; font-family: 'Outfit', sans-serif; font-size: 15px; outline: none; transition: border-color 0.2s; }
    .input-field:focus { border-color: #a855f7; }
    .input-field::placeholder { color: rgba(255,255,255,0.3); }
    .feed-scroll { flex: 1; overflow-y: auto; padding: 12px 16px 100px; }
    .feed-scroll::-webkit-scrollbar { width: 0; }
    .post-card { border-radius: 20px; overflow: hidden; margin-bottom: 20px; background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.06); }
    .post-content-wrap { position: relative; width: 100%; aspect-ratio: 1/1; display: flex; align-items: center; justify-content: center; overflow: hidden; background-size: cover !important; background-position: center !important; }
    .avatar-circle { width: 42px; height: 42px; border-radius: 50%; background: linear-gradient(135deg, #a855f7, #6d28d9); display: flex; align-items: center; justify-content: center; font-weight: 700; font-size: 16px; color: white; overflow: hidden; flex-shrink: 0; position: relative; }
    .avatar-circle img { width: 100%; height: 100%; object-fit: cover; }
    .status-dot { position: absolute; bottom: 0; right: 0; width: 12px; height: 12px; border-radius: 50%; border: 2px solid #0a0a0f; background: #23a55a; }
    .bottom-bar { position: absolute; bottom: 0; left: 0; right: 0; height: 80px; background: linear-gradient(transparent, rgba(10,10,15,0.95) 30%); display: flex; align-items: center; justify-content: center; padding-bottom: 12px; }
    .capture-btn { width: 64px; height: 64px; border-radius: 50%; background: linear-gradient(135deg, #a855f7, #6d28d9); border: 3px solid rgba(255,255,255,0.3); cursor: pointer; display: flex; align-items: center; justify-content: center; transition: all 0.2s; box-shadow: 0 4px 20px rgba(168,85,247,0.4); }
    .capture-btn:hover { transform: scale(1.08); }
    .tab-bar { display: flex; background: rgba(10,10,15,0.95); border-top: 1px solid rgba(255,255,255,0.06); padding: 8px 0 12px; }
    .tab-item { flex: 1; display: flex; flex-direction: column; align-items: center; gap: 4px; cursor: pointer; color: rgba(255,255,255,0.4); font-size: 11px; font-weight: 500; transition: color 0.2s; background: none; border: none; font-family: 'Outfit', sans-serif; position: relative; z-index: 1; }
    .tab-item.active { color: #a855f7; }
    .tab-item i { width: 22px; height: 22px; }
    .toast { position: fixed; top: 20px; left: 50%; transform: translateX(-50%) translateY(-100px); background: #a855f7; color: white; padding: 12px 24px; border-radius: 12px; font-weight: 500; z-index: 9999; transition: transform 0.3s ease; font-size: 14px; }
    .toast.show { transform: translateX(-50%) translateY(0); }
    .toast.error { background: #ef4444; }
    .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.7); z-index: 100; display: none; align-items: center; justify-content: center; padding: 20px; }
    .modal-overlay.active { display: flex; }
    .modal-box { background: #1a1a2e; border: 1px solid rgba(255,255,255,0.1); border-radius: 20px; padding: 24px; width: 100%; max-width: 380px; }
    .friend-item { display: flex; align-items: center; gap: 12px; padding: 12px 16px; background: rgba(255,255,255,0.04); border-radius: 14px; margin-bottom: 8px; }
    @keyframes fadeIn { from { opacity: 0; transform: translateY(16px); } to { opacity: 1; transform: translateY(0); } }
    .fade-in { animation: fadeIn 0.4s ease forwards; }
    
    .discord-call { position:fixed; inset:0; z-index:600; display:none; flex-direction:column; background:#111214; color:#dbdee1; font-family:'Outfit', sans-serif; }
    .discord-call.active { display:flex; }
    .discord-header { padding:16px 20px; background:#1e1f22; display:flex; align-items:center; justify-content:space-between; border-bottom:1px solid #2b2d31; }
    .discord-grid { flex:1; display:grid; grid-template-columns:repeat(auto-fit, minmax(240px, 1fr)); gap:16px; padding:24px; align-content:center; justify-content:center; overflow-y:auto; }
    .discord-user-tile { background:#2b2d31; border-radius:12px; display:flex; flex-direction:column; align-items:center; justify-content:center; aspect-ratio:16/9; position:relative; border:2px solid transparent; }
    .discord-user-tile.speaking { border-color:#23a55a; }
    .discord-controls { padding:20px; background:#1e1f22; display:flex; align-items:center; justify-content:center; gap:16px; border-top:1px solid #2b2d31; }
    .discord-btn { width:52px; height:52px; border-radius:50%; background:#313338; border:none; color:#dbdee1; display:flex; align-items:center; justify-content:center; cursor:pointer; transition:background 0.2s; }
    .discord-btn:hover { background:#35373c; }
    .discord-btn.hangup { background:#ed4245; color:white; }

    .stories-container { display:flex; gap:12px; overflow-x:auto; padding:12px 16px; scrollbar-width:none; border-bottom:1px solid rgba(255,255,255,0.06); background:rgba(255,255,255,0.01); }
    .stories-container::-webkit-scrollbar { display:none; }
    .story-bubble { display:flex; flex-direction:column; align-items:center; gap:4px; cursor:pointer; flex-shrink:0; }
    .story-ring { width:60px; height:60px; border-radius:50%; padding:3px; background:linear-gradient(135deg, #a855f7, #ec4899); display:flex; align-items:center; justify-content:center; }
    .story-inner { width:100%; height:100%; border-radius:50%; background:#1a1a2e; overflow:hidden; display:flex; align-items:center; justify-content:center; }
  </style>
</head>
<body>
  <div class="app-shell gradient-bg" id="app">
    <div class="toast" id="toast"></div>

    <!-- AUTH SCREEN -->
    <div class="screen active" id="screen-auth">
      <div style="flex:1;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:32px;">
        <div style="margin-bottom:40px;text-align:center;">
          <div style="width:72px;height:72px;border-radius:20px;background:linear-gradient(135deg,#a855f7,#6d28d9);display:flex;align-items:center;justify-content:center;margin:0 auto 16px;font-size:32px;">📸</div>
          <h1 id="auth-app-title" style="font-size:28px;font-weight:800;background:linear-gradient(135deg,#a855f7,#e879f9);-webkit-background-clip:text;-webkit-text-fill-color:transparent;">Post-it Social</h1>
          <p style="color:rgba(255,255,255,0.4);font-size:14px;margin-top:6px;">Share moments with friends</p>
        </div>
        <div style="width:100%;max-width:340px;">
          <div id="auth-tabs" style="display:flex;gap:4px;background:rgba(255,255,255,0.06);border-radius:12px;padding:4px;margin-bottom:24px;">
            <button onclick="switchAuthTab('login')" id="tab-login" style="flex:1;padding:10px;border-radius:10px;border:none;font-family:'Outfit';font-weight:600;font-size:14px;cursor:pointer;background:#a855f7;color:white;">Log In</button>
            <button onclick="switchAuthTab('signup')" id="tab-signup" style="flex:1;padding:10px;border-radius:10px;border:none;font-family:'Outfit';font-weight:600;font-size:14px;cursor:pointer;background:transparent;color:rgba(255,255,255,0.5);">Sign Up</button>
          </div>
          <form id="auth-form" onsubmit="handleAuth(event)">
            <div style="display:flex;flex-direction:column;gap:12px;">
              <div>
                <label for="auth-username" style="font-size:13px;color:rgba(255,255,255,0.5);margin-bottom:6px;display:block;">Username</label>
                <input id="auth-username" class="input-field" type="text" placeholder="Enter username" required autocomplete="off">
              </div>
              <div>
                <label for="auth-password" style="font-size:13px;color:rgba(255,255,255,0.5);margin-bottom:6px;display:block;">Password</label>
                <input id="auth-password" class="input-field" type="password" placeholder="Enter password" required>
              </div>
              <p id="auth-error" style="color:#ef4444;font-size:13px;display:none;"></p>
              <button type="submit" class="btn-primary" id="auth-submit-btn">Log In</button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <!-- MAIN FEED SCREEN -->
    <div class="screen" id="screen-feed">
      <header style="padding:14px 20px;display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid rgba(255,255,255,0.06);">
        <h2 id="feed-app-title" style="font-size:20px;font-weight:800;background:linear-gradient(135deg,#a855f7,#e879f9);-webkit-background-clip:text;-webkit-text-fill-color:transparent;margin:0;">Post-it Social</h2>
        <div style="display:flex;gap:8px;">
          <button onclick="showScreen('screen-search')" style="background:rgba(255,255,255,0.08);border:none;border-radius:10px;width:38px;height:38px;display:flex;align-items:center;justify-content:center;cursor:pointer;color:#ccc;"><i data-lucide="user-plus" style="width:18px;height:18px;"></i></button>
        </div>
      </header>
      <!-- Stories Bar -->
      <div class="stories-container" id="stories-container">
        <div class="story-bubble" onclick="openCreateStory()">
          <div class="story-ring" style="background:rgba(255,255,255,0.2);">
            <div class="story-inner"><i data-lucide="plus" style="width:20px;height:20px;color:white;"></i></div>
          </div>
          <span style="font-size:11px;color:rgba(255,255,255,0.7);">Add Story</span>
        </div>
        <div id="stories-list" style="display:flex;gap:12px;"></div>
      </div>
      <div class="feed-scroll" id="feed-container">
        <div id="feed-empty" style="text-align:center;padding:60px 20px;color:rgba(255,255,255,0.3);">
          <i data-lucide="users" style="width:48px;height:48px;margin:0 auto 16px;display:block;"></i>
          <p style="font-size:16px;font-weight:600;margin-bottom:8px;">No posts yet</p>
          <p style="font-size:13px;">Add friends and start sharing moments!</p>
        </div>
        <div id="feed-posts"></div>
      </div>
      <div class="bottom-bar">
        <div class="capture-btn" onclick="openCreatePost()">
          <i data-lucide="plus" style="width:28px;height:28px;color:white;"></i>
        </div>
      </div>
      <div class="tab-bar">
        <button class="tab-item active" onclick="showScreen('screen-feed')"><i data-lucide="home"></i>Feed</button>
        <button class="tab-item" onclick="showScreen('screen-friends')"><i data-lucide="users"></i>Friends</button>
        <button class="tab-item" onclick="showScreen('screen-messages')"><i data-lucide="message-square"></i>Messages</button>
        <button class="tab-item" onclick="showScreen('screen-profile')"><i data-lucide="user"></i>Profile</button>
      </div>
    </div>

    <!-- SEARCH SCREEN -->
    <div class="screen" id="screen-search">
      <header style="padding:14px 20px;display:flex;align-items:center;gap:12px;border-bottom:1px solid rgba(255,255,255,0.06);">
        <button onclick="showScreen('screen-feed')" style="background:none;border:none;color:#ccc;cursor:pointer;"><i data-lucide="arrow-left" style="width:22px;height:22px;"></i></button>
        <h2 style="font-size:18px;font-weight:700;">Add Friend</h2>
      </header>
      <div style="padding:16px 20px;flex:1;overflow-y:auto;display:flex;flex-direction:column;">
        <div style="position:relative;margin-bottom:16px;">
          <i data-lucide="search" style="position:absolute;left:14px;top:50%;transform:translateY(-50%);width:18px;height:18px;color:rgba(255,255,255,0.3);"></i>
          <input id="search-input" class="input-field" style="padding-left:42px;" type="text" placeholder="Search by username..." oninput="handleSearch(this.value)">
        </div>
        <div id="search-results" style="flex:1;"></div>
      </div>
    </div>

    <!-- FRIENDS SCREEN -->
    <div class="screen" id="screen-friends">
      <header style="padding:14px 20px;display:flex;align-items:center;gap:12px;border-bottom:1px solid rgba(255,255,255,0.06);">
        <h2 style="font-size:18px;font-weight:700;">Friends</h2>
      </header>
      <div style="flex:1;overflow-y:auto;padding:16px 20px;" id="friends-list-container">
        <div id="friends-empty" style="text-align:center;padding:40px 20px;color:rgba(255,255,255,0.3);">
          <p style="font-size:14px;">No friends yet. Search to add some!</p>
        </div>
        <div id="friends-list"></div>
      </div>
      <div class="tab-bar">
        <button class="tab-item" onclick="showScreen('screen-feed')"><i data-lucide="home"></i>Feed</button>
        <button class="tab-item active" onclick="showScreen('screen-friends')"><i data-lucide="users"></i>Friends</button>
        <button class="tab-item" onclick="showScreen('screen-messages')"><i data-lucide="message-square"></i>Messages</button>
        <button class="tab-item" onclick="showScreen('screen-profile')"><i data-lucide="user"></i>Profile</button>
      </div>
    </div>

    <!-- MESSAGES SCREEN -->
    <div class="screen" id="screen-messages">
      <div style="display:flex;height:100%;width:100%;overflow:hidden;">
        <div id="dm-sidebar" style="width:32%;background:rgba(255,255,255,0.02);border-right:1px solid rgba(255,255,255,0.06);overflow-y:auto;display:flex;flex-direction:column;">
          <header style="padding:14px 16px;border-bottom:1px solid rgba(255,255,255,0.06);display:flex;align-items:center;justify-content:space-between;flex-shrink:0;">
            <h2 style="font-size:18px;font-weight:700;margin:0;">Chats</h2>
            <button onclick="openGroupModal()" class="btn-secondary" style="padding:6px 10px;font-size:11px;">+ Group</button>
          </header>
          <div id="dm-chats-list" style="flex:1;overflow-y:auto;padding:8px;">
            <div id="dm-empty-state" style="text-align:center;padding:40px 16px;color:rgba(255,255,255,0.3);font-size:13px;">No conversations yet</div>
          </div>
        </div>
        <div id="dm-chat-area" style="flex:1;display:none;flex-direction:column;background:#0a0a0f;overflow:hidden;">
          <header style="padding:14px 20px;border-bottom:1px solid rgba(255,255,255,0.06);display:flex;align-items:center;gap:12px;flex-shrink:0;">
            <button onclick="closeDmChat()" style="background:none;border:none;color:#ccc;cursor:pointer;"><i data-lucide="arrow-left" style="width:22px;height:22px;"></i></button>
            <h2 id="dm-header-name" style="font-size:16px;font-weight:700;margin:0;flex:1;">Chat</h2>
            <button id="call-trigger-btn" onclick="startCurrentChatCall()" class="btn-secondary" style="padding:8px 12px;display:flex;align-items:center;gap:6px;font-size:12px;"><i data-lucide="phone" style="width:15px;height:15px;"></i><span>Call</span></button>
          </header>
          <div id="dm-messages-container" style="flex:1;overflow-y:auto;padding:16px 20px;display:flex;flex-direction:column;gap:12px;"></div>
          <div style="padding:16px 20px;border-top:1px solid rgba(255,255,255,0.06);display:flex;gap:8px;flex-shrink:0;">
            <input id="dm-input" type="text" placeholder="Type a message..." style="flex:1;padding:12px 14px;background:rgba(255,255,255,0.06);border:1px solid rgba(255,255,255,0.12);border-radius:10px;color:#f0f0f0;font-family:'Outfit';font-size:14px;outline:none;" maxlength="200">
            <button onclick="sendDmMessage()" class="btn-primary" style="padding:12px 20px;width:auto;min-width:70px;height:auto;">Send</button>
          </div>
        </div>
      </div>
      <div class="tab-bar">
        <button class="tab-item" onclick="showScreen('screen-feed')"><i data-lucide="home"></i>Feed</button>
        <button class="tab-item" onclick="showScreen('screen-friends')"><i data-lucide="users"></i>Friends</button>
        <button class="tab-item active" onclick="showScreen('screen-messages')"><i data-lucide="message-square"></i>Messages</button>
        <button class="tab-item" onclick="showScreen('screen-profile')"><i data-lucide="user"></i>Profile</button>
      </div>
    </div>

    <!-- PROFILE SCREEN -->
    <div class="screen" id="screen-profile">
      <header style="padding:14px 20px;display:flex;align-items:center;gap:12px;border-bottom:1px solid rgba(255,255,255,0.06);">
        <h2 style="font-size:18px;font-weight:700;">Profile</h2>
      </header>
      <div style="flex:1;overflow-y:auto;padding:24px 20px;">
        <div style="text-align:center;margin-bottom:24px;">
          <div id="profile-avatar" class="avatar-circle" style="width:80px;height:80px;font-size:32px;margin:0 auto 8px;cursor:pointer;position:relative;overflow:hidden;flex-shrink:0;" onclick="openAvatarModal()">
            <span id="profile-avatar-emoji">😊</span>
          </div>
          <p style="font-size:12px;color:rgba(255,255,255,0.3);margin-top:4px;">Tap avatar to customize</p>
          <h3 id="profile-display-name" style="font-size:22px;font-weight:700;margin-top:8px;"></h3>
          <p id="profile-username" style="font-size:14px;color:rgba(255,255,255,0.5);margin-top:2px;"></p>
          <p id="profile-bio" style="font-size:14px;color:rgba(255,255,255,0.8);margin-top:10px;font-style:italic;"></p>
          <p id="profile-status" style="font-size:13px;color:#23a55a;margin-top:6px;font-weight:500;"></p>
        </div>
        <div style="display:flex;flex-direction:column;gap:12px;margin-bottom:24px;">
          <button onclick="openEditProfileModal()" class="btn-secondary" style="width:100%;">Edit Display Name & Bio</button>
          <button onclick="openStatusModal()" class="btn-secondary" style="width:100%;">Set Custom Status</button>
        </div>
        <div style="display:flex;flex-direction:column;gap:8px;">
          <div class="card-glass" style="padding:16px;display:flex;align-items:center;justify-content:space-between;">
            <span style="font-size:14px;color:rgba(255,255,255,0.6);">My Posts</span>
            <span id="profile-post-count" style="font-weight:700;color:#a855f7;">0</span>
          </div>
          <div class="card-glass" style="padding:16px;display:flex;align-items:center;justify-content:space-between;">
            <span style="font-size:14px;color:rgba(255,255,255,0.6);">Friends</span>
            <span id="profile-friend-count" style="font-weight:700;color:#a855f7;">0</span>
          </div>
        </div>
        <button onclick="handleLogout()" class="btn-secondary" style="width:100%;margin-top:24px;color:#ef4444;border-color:rgba(239,68,68,0.3);">Log Out</button>
      </div>
      <div class="tab-bar">
        <button class="tab-item" onclick="showScreen('screen-feed')"><i data-lucide="home"></i>Feed</button>
        <button class="tab-item" onclick="showScreen('screen-friends')"><i data-lucide="users"></i>Friends</button>
        <button class="tab-item" onclick="showScreen('screen-messages')"><i data-lucide="message-square"></i>Messages</button>
        <button class="tab-item active" onclick="showScreen('screen-profile')"><i data-lucide="user"></i>Profile</button>
      </div>
    </div>

    <!-- CREATE POST / STORY SCREEN -->
    <div class="screen" id="screen-camera">
      <header style="padding:14px 20px;display:flex;align-items:center;gap:12px;border-bottom:1px solid rgba(255,255,255,0.06);">
        <button onclick="showScreen('screen-feed')" style="background:none;border:none;color:#ccc;cursor:pointer;"><i data-lucide="arrow-left" style="width:22px;height:22px;"></i></button>
        <h2 id="create-mode-title" style="font-size:18px;font-weight:700;">Create Post</h2>
      </header>
      <div style="flex:1;overflow-y:auto;padding:16px 20px;display:flex;flex-direction:column;gap:16px;">
        <div style="display:flex;flex-direction:column;gap:8px;">
          <label style="font-size:13px;color:rgba(255,255,255,0.5);">Text Overlays</label>
          <div id="text-items-container" style="display:flex;flex-direction:column;gap:8px;margin-bottom:12px;"></div>
          <button type="button" onclick="addTextItem()" class="btn-secondary" style="width:100%;">+ Add Text</button>
        </div>
        <div style="display:flex;flex-direction:column;gap:12px;">
          <p style="font-size:13px;color:rgba(255,255,255,0.5);">Background Style</p>
          <div style="display:flex;gap:8px;">
            <button onclick="selectBackgroundType('solid')" id="btn-solid" style="flex:1;padding:12px;border-radius:10px;border:2px solid #a855f7;background:rgba(168,85,247,0.2);color:#a855f7;font-family:'Outfit';font-weight:600;font-size:13px;cursor:pointer;">Solid</button>
            <button onclick="selectBackgroundType('gradient')" id="btn-gradient" style="flex:1;padding:12px;border-radius:10px;border:2px solid rgba(255,255,255,0.1);background:transparent;color:rgba(255,255,255,0.6);font-family:'Outfit';font-weight:600;font-size:13px;cursor:pointer;">Gradient</button>
            <button onclick="selectBackgroundType('photo')" id="btn-photo" style="flex:1;padding:12px;border-radius:10px;border:2px solid rgba(255,255,255,0.1);background:transparent;color:rgba(255,255,255,0.6);font-family:'Outfit';font-weight:600;font-size:13px;cursor:pointer;">Photo</button>
          </div>
        </div>
        <div id="bg-controls-container"></div>
        <div style="padding-top:16px;">
          <button onclick="publishContent()" id="publish-thought-btn" class="btn-primary">Share</button>
        </div>
      </div>
      <div style="padding:16px;border-top:1px solid rgba(255,255,255,0.06);">
        <div id="preview-box" style="width:100%;max-width:260px;margin:0 auto;aspect-ratio:1/1;border-radius:16px;display:flex;align-items:center;justify-content:center;background:#4a5568;position:relative;overflow:hidden;background-size:cover !important;background-position:center !important;">
          <p style="text-align:center;color:rgba(255,255,255,0.5);padding:20px;font-size:14px;">Preview</p>
        </div>
      </div>
    </div>

    <!-- VIEW STORY SCREEN -->
    <div class="screen" id="screen-view-story" style="background:#000;justify-content:center;align-items:center;position:relative;">
      <button onclick="showScreen('screen-feed')" style="position:absolute;top:20px;right:20px;background:rgba(0,0,0,0.5);border:none;color:white;border-radius:50%;width:40px;height:40px;font-size:20px;cursor:pointer;z-index:10;">✕</button>
      <div id="story-viewer-content" style="width:100%;max-width:400px;aspect-ratio:9/16;position:relative;background-size:cover;background-position:center;display:flex;flex-direction:column;justify-content:space-between;padding:24px;"></div>
    </div>

    <!-- USER PROFILE VIEW SCREEN -->
    <div class="screen" id="screen-user-profile">
      <header style="padding:14px 20px;display:flex;align-items:center;gap:12px;border-bottom:1px solid rgba(255,255,255,0.06);">
        <button onclick="closeUserProfile()" style="background:none;border:none;color:#ccc;cursor:pointer;"><i data-lucide="arrow-left" style="width:22px;height:22px;"></i></button>
        <h2 id="user-profile-header" style="font-size:18px;font-weight:700;"></h2>
      </header>
      <div style="flex:1;overflow-y:auto;padding:24px 20px;">
        <div style="text-align:center;margin-bottom:24px;">
          <div id="user-profile-avatar" class="avatar-circle" style="width:80px;height:80px;font-size:32px;margin:0 auto 8px;position:relative;overflow:hidden;flex-shrink:0;">
            <span id="user-profile-avatar-emoji">😊</span>
          </div>
          <h3 id="user-profile-display-name" style="font-size:22px;font-weight:700;margin-top:8px;"></h3>
          <p id="user-profile-username" style="font-size:14px;color:rgba(255,255,255,0.5);margin-top:2px;"></p>
          <p id="user-profile-bio" style="font-size:14px;color:rgba(255,255,255,0.8);margin-top:10px;font-style:italic;"></p>
          <p id="user-profile-status" style="font-size:13px;color:#23a55a;margin-top:6px;font-weight:500;"></p>
        </div>
        <div style="display:flex;flex-direction:column;gap:8px;">
          <div class="card-glass" style="padding:16px;display:flex;align-items:center;justify-content:space-between;">
            <span style="font-size:14px;color:rgba(255,255,255,0.6);">Posts</span>
            <span id="user-profile-post-count" style="font-weight:700;color:#a855f7;">0</span>
          </div>
          <div class="card-glass" style="padding:16px;display:flex;align-items:center;justify-content:space-between;">
            <span style="font-size:14px;color:rgba(255,255,255,0.6);">Friends</span>
            <span id="user-profile-friend-count" style="font-weight:700;color:#a855f7;">0</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- AVATAR MODAL -->
  <div class="modal-overlay" id="avatar-modal">
    <div class="modal-box" style="display:flex;flex-direction:column;max-width:400px;max-height:80vh;">
      <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:20px;">
        <h3 style="font-size:18px;font-weight:700;margin:0;">Customize Avatar</h3>
        <button onclick="closeAvatarModal()" style="background:none;border:none;color:#ccc;cursor:pointer;font-size:20px;">✕</button>
      </div>
      <div id="avatar-preview" style="width:120px;height:120px;border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:56px;background:#a855f7;margin:0 auto 24px;overflow:hidden;background-size:cover !important;background-position:center !important;">😊</div>
      <label class="btn-secondary" style="display:block;text-align:center;margin-bottom:14px;cursor:pointer;width:100%;">
        📷 Upload Photo
        <input type="file" accept="image/*" style="display:none;" onchange="handleAvatarPhotoUpload(event)">
      </label>
      <div id="emoji-picker" style="display:grid;grid-template-columns:repeat(4,1fr);gap:4px;max-height:160px;overflow-y:auto;margin-bottom:16px;"></div>
      <button onclick="saveAvatarCustomization()" class="btn-primary" style="width:100%;">Save Avatar</button>
    </div>
  </div>

  <!-- EDIT PROFILE MODAL -->
  <div class="modal-overlay" id="edit-profile-modal">
    <div class="modal-box">
      <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:16px;">
        <h3 style="font-size:18px;font-weight:700;margin:0;">Edit Profile</h3>
        <button onclick="closeEditProfileModal()" style="background:none;border:none;color:#ccc;cursor:pointer;font-size:20px;">✕</button>
      </div>
      <div style="display:flex;flex-direction:column;gap:12px;">
        <div>
          <label style="font-size:12px;color:rgba(255,255,255,0.5);margin-bottom:4px;display:block;">Display Name</label>
          <input id="edit-display-name-input" class="input-field" type="text" placeholder="Your display name">
        </div>
        <div>
          <label style="font-size:12px;color:rgba(255,255,255,0.5);margin-bottom:4px;display:block;">Bio</label>
          <textarea id="edit-bio-input" class="input-field" rows="3" placeholder="Tell us about yourself..." style="resize:none;" maxlength="150"></textarea>
        </div>
        <button onclick="saveProfileEdits()" class="btn-primary" style="margin-top:8px;">Save Changes</button>
      </div>
    </div>
  </div>

  <!-- CUSTOM STATUS MODAL -->
  <div class="modal-overlay" id="status-modal">
    <div class="modal-box">
      <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:16px;">
        <h3 style="font-size:18px;font-weight:700;margin:0;">Set Status</h3>
        <button onclick="closeStatusModal()" style="background:none;border:none;color:#ccc;cursor:pointer;font-size:20px;">✕</button>
      </div>
      <div style="display:flex;flex-direction:column;gap:12px;">
        <input id="custom-status-input" class="input-field" type="text" placeholder="e.g. Playing Valorant, Listening to Music..." maxlength="60">
        <button onclick="saveCustomStatus()" class="btn-primary">Update Status</button>
      </div>
    </div>
  </div>

  <!-- GROUP CHAT MODAL -->
  <div class="modal-overlay" id="group-modal">
    <div class="modal-box">
      <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:16px;">
        <h3 style="font-size:18px;font-weight:700;margin:0;">Create Group Chat</h3>
        <button onclick="closeGroupModal()" style="background:none;border:none;color:#ccc;cursor:pointer;font-size:20px;">✕</button>
      </div>
      <div style="display:flex;flex-direction:column;gap:12px;">
        <input id="group-name-input" class="input-field" type="text" placeholder="Group Name">
        <p style="font-size:12px;color:rgba(255,255,255,0.5);">Select Friends:</p>
        <div id="group-friends-checkboxes" style="max-height:160px;overflow-y:auto;display:flex;flex-direction:column;gap:6px;"></div>
        <button onclick="createGroupChat()" class="btn-primary" style="margin-top:8px;">Create Group</button>
      </div>
    </div>
  </div>

  <!-- DISCORD-STYLE VOICE CALL SCREEN -->
  <div class="discord-call" id="discord-call-screen">
    <div class="discord-header">
      <div style="display:flex;align-items:center;gap:10px;">
        <div style="width:10px;height:10px;border-radius:50%;background:#23a55a;"></div>
        <h3 id="discord-call-title" style="font-size:16px;font-weight:600;margin:0;">Voice Channel</h3>
      </div>
      <span id="discord-call-timer" style="font-size:13px;color:#949ba4;">00:00</span>
    </div>
    <div id="discord-grid" class="discord-grid"></div>
    <div class="discord-controls">
      <button class="discord-btn" onclick="toggleDiscordMute()"><i data-lucide="mic" id="discord-mic-icon"></i></button>
      <button class="discord-btn hangup" onclick="endDiscordCall()"><i data-lucide="phone-off"></i></button>
      <button class="discord-btn" onclick="toggleDiscordDeafen()"><i data-lucide="headphones" id="discord-deaf-icon"></i></button>
    </div>
  </div>

  <script>
// ===== AUTO-PRUNING STORAGE (PREVENTS STORAGE FULL ERRORS) =====
window.elementSdk = window.elementSdk || { init() { return Promise.resolve({ isOk: true }); } };
window.dataSdk = window.dataSdk || (function () {
  const STORAGE_KEY = 'postit_social_optimized_v4';
  let handler = null;
  function load() {
    try { return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'); }
    catch (e) { return []; }
  }
  function save(data) {
    try {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
      if (handler && handler.onDataChanged) handler.onDataChanged(data);
      return true;
    } catch (e) {
      // Auto-prune oldest posts & stories to reclaim space if quota exceeded
      try {
        let posts = data.filter(d => d.type === 'post' || d.type === 'story');
        if (posts.length > 5) {
          posts.sort((a,b) => new Date(a.post_date || a.story_date) - new Date(b.post_date || b.story_date));
          const removeIds = [posts[0].__backendId];
          data = data.filter(d => !removeIds.includes(d.__backendId));
          localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
          if (handler && handler.onDataChanged) handler.onDataChanged(data);
          return true;
        }
      } catch (err) {}
      alert('Storage limit reached. Oldest items pruned automatically.');
      return false;
    }
  }
  return {
    init(h) {
      handler = h;
      setTimeout(() => { if (handler && handler.onDataChanged) handler.onDataChanged(load()); }, 0);
      return Promise.resolve({ isOk: true });
    },
    create(item) {
      const data = load();
      const record = { ...item, __backendId: 'id_' + Date.now() + '_' + Math.random().toString(36).slice(2, 7) };
      data.push(record);
      return Promise.resolve({ isOk: save(data), data: record });
    },
    update(updatedItem) {
      const data = load();
      const idx = data.findIndex(d => d.__backendId === updatedItem.__backendId);
      if (idx === -1) return Promise.resolve({ isOk: false });
      data[idx] = { ...updatedItem };
      return Promise.resolve({ isOk: save(data) });
    },
    delete(targetItem) {
      const data = load().filter(d => d.__backendId !== targetItem.__backendId);
      return Promise.resolve({ isOk: save(data) });
    }
  };
})();

// ===== APP STATE =====
let allData = [];
let currentUser = null;
let viewingUserProfile = null;
let authMode = 'login';
let textItems = [];
let currentChatId = null;
let createMode = 'post';

let bgType = 'solid';
let bgColor1 = '#a855f7';
let bgColor2 = '#6d28d9';
let bgPhotoDataUrl = '';

let avatarEmoji = '😊';
const emojiList = ['😊', '😄', '🤔', '😎', '🥳', '😍', '🤗', '😌', '🎨', '🎭', '🎪', '🎸', '🎤', '🎬', '🎮', '⚡', '🔥', '💎', '👑', '🌟', '✨', '🚀', '🌈', '🎯', '🏆', '💪', '👍', '🤝', '💯'];
const defaultConfig = { app_title: 'Post-it Social' };

// Ultra-aggressive compression so images take minimal space
function compressImageFile(file, maxSize = 160, quality = 0.5) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => {
        const canvas = document.createElement('canvas');
        let w = img.width, h = img.height;
        if (w > h && w > maxSize) { h = Math.round((h * maxSize) / w); w = maxSize; }
        else if (h >= w && h > maxSize) { w = Math.round((w * maxSize) / h); h = maxSize; }
        canvas.width = w; canvas.height = h;
        canvas.getContext('2d').drawImage(img, 0, 0, w, h);
        resolve(canvas.toDataURL('image/jpeg', quality));
      };
      img.onerror = reject;
      img.src = e.target.result;
    };
    reader.onerror = reject;
    reader.readAsDataURL(file);
  });
}

window.elementSdk.init({
  defaultConfig,
  onConfigChange: async (config) => {
    const t = config.app_title || defaultConfig.app_title;
    document.getElementById('auth-app-title').textContent = t;
    document.getElementById('feed-app-title').textContent = t;
  }
});

const dataHandler = {
  onDataChanged(data) {
    allData = data;
    const savedUser = localStorage.getItem('locket_user');
    if (savedUser && !currentUser) {
      const parsed = JSON.parse(savedUser);
      const userRec = allData.find(d => d.type === 'user' && d.username === parsed.username);
      if (userRec) {
        currentUser = {
          username: userRec.username,
          __backendId: userRec.__backendId,
          display_name: userRec.display_name || userRec.username,
          bio: userRec.bio || '',
          status: userRec.status || 'Online',
          avatar_emoji: userRec.avatar_emoji || '😊'
        };
        avatarEmoji = currentUser.avatar_emoji;
        showScreen('screen-feed');
      }
    }
    renderFeed();
    renderStories();
    renderFriendsList();
    renderChatsList();
    if (viewingUserProfile) renderUserProfile(viewingUserProfile);
    else renderProfile();
  }
};

(async () => { await window.dataSdk.init(dataHandler); })();

function toast(msg, isError) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.className = 'toast show' + (isError ? ' error' : '');
  setTimeout(() => t.className = 'toast', 2500);
}

function showScreen(id) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  if (['screen-feed','screen-friends','screen-messages','screen-profile'].includes(id)) {
    document.querySelectorAll('.tab-bar').forEach(bar => {
      bar.querySelectorAll('.tab-item').forEach((t, i) => {
        const tabs = ['screen-feed','screen-friends','screen-messages','screen-profile'];
        t.classList.toggle('active', tabs[i] === id);
      });
    });
  }
  lucide.createIcons();
}

function getUsers() { return allData.filter(d => d.type === 'user'); }
function getUserRecord(username) { return getUsers().find(u => u.username === username); }
function getFriends() { return allData.filter(d => d.type === 'friend' && (d.friend_of === currentUser?.username || d.friend_with === currentUser?.username)); }
function getMyFriendUsernames() { return getFriends().map(f => f.friend_of === currentUser?.username ? f.friend_with : f.friend_of); }
function getPosts() { return allData.filter(d => d.type === 'post'); }
function getStories() { return allData.filter(d => d.type === 'story'); }
function getGroups() { return allData.filter(d => d.type === 'group' && (d.group_members || []).includes(currentUser?.username)); }

function escapeHtml(str) {
  const d = document.createElement('div');
  d.textContent = str || '';
  return d.innerHTML;
}

function avatarHTML(username, size = 42) {
  const user = getUserRecord(username);
  const av = user?.avatar_emoji || '😊';
  let inner = av.startsWith('data:image') ? `<img src="${av}" alt="@${username}">` : `<span style="font-size:${size*0.45}px;">${av}</span>`;
  return `<div class="avatar-circle" style="width:${size}px;height:${size}px;">${inner}<div class="status-dot"></div></div>`;
}

// ===== AUTH =====
function switchAuthTab(mode) {
  authMode = mode;
  document.getElementById('tab-login').style.background = mode === 'login' ? '#a855f7' : 'transparent';
  document.getElementById('tab-login').style.color = mode === 'login' ? 'white' : 'rgba(255,255,255,0.5)';
  document.getElementById('tab-signup').style.background = mode === 'signup' ? '#a855f7' : 'transparent';
  document.getElementById('tab-signup').style.color = mode === 'signup' ? 'white' : 'rgba(255,255,255,0.5)';
  document.getElementById('auth-submit-btn').textContent = mode === 'login' ? 'Log In' : 'Sign Up';
  document.getElementById('auth-error').style.display = 'none';
}

async function handleAuth(e) {
  e.preventDefault();
  const username = document.getElementById('auth-username').value.trim().toLowerCase();
  const password = document.getElementById('auth-password').value;
  const errEl = document.getElementById('auth-error');
  if (!username || !password) { errEl.textContent = 'Fill in all fields'; errEl.style.display = 'block'; return; }

  const hash = 'h_' + password;
  const existing = getUsers().find(u => u.username === username);

  if (authMode === 'signup') {
    if (existing) { errEl.textContent = 'Username taken'; errEl.style.display = 'block'; return; }
    const r = await window.dataSdk.create({ type: 'user', username, password_hash: hash, display_name: username, bio: 'Hey there!', status: 'Online', avatar_emoji: '😊' });
    if (r.isOk) {
      currentUser = { username, __backendId: r.data.__backendId, display_name: username, bio: 'Hey there!', status: 'Online', avatar_emoji: '😊' };
      localStorage.setItem('locket_user', JSON.stringify({ username }));
      toast('Account created! 🎉');
      showScreen('screen-feed');
    }
  } else {
    if (!existing || existing.password_hash !== hash) { errEl.textContent = 'Invalid credentials'; errEl.style.display = 'block'; return; }
    currentUser = { username: existing.username, __backendId: existing.__backendId, display_name: existing.display_name || username, bio: existing.bio || '', status: existing.status || 'Online', avatar_emoji: existing.avatar_emoji || '😊' };
    localStorage.setItem('locket_user', JSON.stringify({ username }));
    toast('Welcome back! 👋');
    showScreen('screen-feed');
  }
}

function handleLogout() {
  currentUser = null;
  localStorage.removeItem('locket_user');
  showScreen('screen-auth');
  toast('Logged out');
}

// ===== PROFILE & EDIT =====
function renderProfile() {
  if (!currentUser) return;
  const userRec = getUserRecord(currentUser.username);
  if (!userRec) return;
  document.getElementById('profile-display-name').textContent = userRec.display_name || userRec.username;
  document.getElementById('profile-username').textContent = '@' + userRec.username;
  document.getElementById('profile-bio').textContent = userRec.bio || 'No bio yet.';
  document.getElementById('profile-status').textContent = '● ' + (userRec.status || 'Online');
  
  const avEl = document.getElementById('profile-avatar-emoji');
  if (userRec.avatar_emoji.startsWith('data:image')) {
    avEl.innerHTML = `<img src="${userRec.avatar_emoji}" style="width:100%;height:100%;object-fit:cover;border-radius:50%;">`;
  } else {
    avEl.textContent = userRec.avatar_emoji;
  }
  document.getElementById('profile-post-count').textContent = getPosts().filter(p => p.post_owner === currentUser.username).length;
  document.getElementById('profile-friend-count').textContent = getMyFriendUsernames().length;
}

function openEditProfileModal() {
  document.getElementById('edit-display-name-input').value = currentUser.display_name;
  document.getElementById('edit-bio-input').value = currentUser.bio || '';
  document.getElementById('edit-profile-modal').classList.add('active');
}
function closeEditProfileModal() { document.getElementById('edit-profile-modal').classList.remove('active'); }

async function saveProfileEdits() {
  const newName = document.getElementById('edit-display-name-input').value.trim();
  const newBio = document.getElementById('edit-bio-input').value.trim();
  if (!newName) { toast('Display name required', true); return; }
  const userRec = getUserRecord(currentUser.username);
  const r = await window.dataSdk.update({ ...userRec, display_name: newName, bio: newBio });
  if (r.isOk) {
    currentUser.display_name = newName;
    currentUser.bio = newBio;
    toast('Profile updated!');
    closeEditProfileModal();
    renderProfile();
  }
}

function openStatusModal() { document.getElementById('status-modal').classList.add('active'); }
function closeStatusModal() { document.getElementById('status-modal').classList.remove('active'); }
async function saveCustomStatus() {
  const statusText = document.getElementById('custom-status-input').value.trim() || 'Online';
  const userRec = getUserRecord(currentUser.username);
  const r = await window.dataSdk.update({ ...userRec, status: statusText });
  if (r.isOk) {
    currentUser.status = statusText;
    toast('Status updated!');
    closeStatusModal();
    renderProfile();
  }
}

function openAvatarModal() {
  document.getElementById('emoji-picker').innerHTML = emojiList.map(e => `<button onclick="selectAvatarEmoji('${e}')" style="background:none;border:none;font-size:32px;cursor:pointer;padding:8px;">${e}</button>`).join('');
  document.getElementById('avatar-modal').classList.add('active');
}
function closeAvatarModal() { document.getElementById('avatar-modal').classList.remove('active'); }
function selectAvatarEmoji(e) {
  avatarEmoji = e;
  const prev = document.getElementById('avatar-preview');
  prev.style.background = '#a855f7';
  prev.textContent = e;
}
async function handleAvatarPhotoUpload(e) {
  const file = e.target.files[0];
  if (!file) return;
  toast('Compressing photo...');
  avatarEmoji = await compressImageFile(file, 160, 0.5);
  const prev = document.getElementById('avatar-preview');
  prev.innerHTML = `<img src="${avatarEmoji}" style="width:100%;height:100%;object-fit:cover;border-radius:50%;">`;
}
async function saveAvatarCustomization() {
  const userRec = getUserRecord(currentUser.username);
  const r = await window.dataSdk.update({ ...userRec, avatar_emoji: avatarEmoji });
  if (r.isOk) {
    currentUser.avatar_emoji = avatarEmoji;
    toast('Avatar saved!');
    closeAvatarModal();
    renderProfile();
  }
}

// ===== SEARCH & FRIENDS =====
function handleSearch(q) {
  const container = document.getElementById('search-results');
  if (!q.trim()) { container.innerHTML = ''; return; }
  const users = getUsers().filter(u => u.username !== currentUser?.username && u.username.includes(q.toLowerCase()));
  const friendNames = getMyFriendUsernames();
  container.innerHTML = users.map(u => `
    <div class="friend-item">
      ${avatarHTML(u.username, 40)}
      <div style="flex:1;"><p style="font-weight:600;font-size:14px;margin:0;">${escapeHtml(u.display_name || u.username)}</p><span style="font-size:12px;color:rgba(255,255,255,0.4);">@${u.username}</span></div>
      ${friendNames.includes(u.username) ? '<span style="font-size:12px;color:#a855f7;">Friends ✓</span>' : `<button class="btn-primary" style="padding:6px 14px;font-size:12px;width:auto;" onclick="addFriend('${u.username}')">Add</button>`}
    </div>
  `).join('');
}

async function addFriend(uname) {
  const r = await window.dataSdk.create({ type: 'friend', friend_of: currentUser.username, friend_with: uname, friend_status: 'active' });
  if (r.isOk) { toast(`Added @${uname}!`); handleSearch(document.getElementById('search-input').value); }
}

function renderFriendsList() {
  if (!currentUser) return;
  const friends = getMyFriendUsernames();
  document.getElementById('friends-empty').style.display = friends.length ? 'none' : 'block';
  document.getElementById('friends-list').innerHTML = friends.map(f => {
    const u = getUserRecord(f);
    return `
      <div class="friend-item">
        ${avatarHTML(f, 42)}
        <div style="flex:1;cursor:pointer;" onclick="viewUserProfile('${f}')">
          <p style="font-weight:600;font-size:14px;margin:0;">${escapeHtml(u?.display_name || f)}</p>
          <span style="font-size:11px;color:#23a55a;">● ${escapeHtml(u?.status || 'Online')}</span>
        </div>
        <button class="btn-secondary" style="padding:6px 12px;font-size:12px;" onclick="openDmChat('${f}')">Message</button>
      </div>
    `;
  }).join('');
}

function viewUserProfile(uname) {
  viewingUserProfile = uname;
  const u = getUserRecord(uname);
  if (!u) return;
  document.getElementById('user-profile-header').textContent = '@' + uname;
  document.getElementById('user-profile-display-name').textContent = u.display_name || uname;
  document.getElementById('user-profile-username').textContent = '@' + uname;
  document.getElementById('user-profile-bio').textContent = u.bio || '';
  document.getElementById('user-profile-status').textContent = '● ' + (u.status || 'Online');
  
  const avEl = document.getElementById('user-profile-avatar');
  if (u.avatar_emoji.startsWith('data:image')) {
    avEl.innerHTML = `<img src="${u.avatar_emoji}" style="width:100%;height:100%;object-fit:cover;border-radius:50%;">`;
  } else {
    avEl.innerHTML = `<span style="font-size:32px;">${u.avatar_emoji}</span>`;
  }
  document.getElementById('user-profile-post-count').textContent = getPosts().filter(p => p.post_owner === uname).length;
  document.getElementById('user-profile-friend-count').textContent = allData.filter(d => d.type === 'friend' && (d.friend_of === uname || d.friend_with === uname)).length;
  showScreen('screen-user-profile');
}
function closeUserProfile() { viewingUserProfile = null; showScreen('screen-feed'); }

// ===== POSTS & STORIES CREATOR =====
function openCreatePost() {
  createMode = 'post';
  document.getElementById('create-mode-title').textContent = 'Create Post';
  showScreen('screen-camera');
  bgType = 'solid'; bgColor1 = '#a855f7'; bgColor2 = '#6d28d9'; bgPhotoDataUrl = '';
  textItems = [];
  renderTextItems();
  selectBackgroundType('solid');
  updatePreview();
}

function openCreateStory() {
  createMode = 'story';
  document.getElementById('create-mode-title').textContent = 'Create Story';
  showScreen('screen-camera');
  bgType = 'solid'; bgColor1 = '#ec4899'; bgColor2 = '#8b5cf6'; bgPhotoDataUrl = '';
  textItems = [];
  renderTextItems();
  selectBackgroundType('solid');
  updatePreview();
}

function addTextItem() {
  textItems.push({ id: 't_' + Date.now(), text: '', x: 50, y: 50, size: 28, color: '#ffffff' });
  renderTextItems();
  updatePreview();
}
function removeTextItem(id) { textItems = textItems.filter(t => t.id !== id); renderTextItems(); updatePreview(); }
function updateTextItem(id, field, val) {
  const t = textItems.find(x => x.id === id);
  if (t) { t[field] = val; updatePreview(); }
}

function renderTextItems() {
  document.getElementById('text-items-container').innerHTML = textItems.map(item => `
    <div style="background:rgba(255,255,255,0.06);border-radius:12px;padding:12px;display:flex;flex-direction:column;gap:8px;">
      <div style="display:flex;justify-content:space-between;"><span style="font-size:12px;color:#aaa;">Text</span><button onclick="removeTextItem('${item.id}')" style="background:none;border:none;color:#ef4444;cursor:pointer;">✕</button></div>
      <textarea oninput="updateTextItem('${item.id}', 'text', this.value)" class="input-field" rows="2" style="font-size:13px;resize:none;">${escapeHtml(item.text)}</textarea>
    </div>
  `).join('');
}

function updatePreview() {
  const box = document.getElementById('preview-box');
  box.style.background = bgType === 'photo' && bgPhotoDataUrl ? `url('${bgPhotoDataUrl}')` : (bgType === 'solid' ? bgColor1 : `linear-gradient(135deg, ${bgColor1}, ${bgColor2})`);
  box.innerHTML = textItems.map(i => `<div style="position:absolute;left:${i.x}%;top:${i.y}%;transform:translate(-50%,-50%);color:${i.color};font-size:${i.size}px;font-weight:700;text-shadow:0 2px 8px rgba(0,0,0,0.5);text-align:center;">${escapeHtml(i.text)}</div>`).join('') || '<p style="color:rgba(255,255,255,0.5);">Preview</p>';
}

function selectBackgroundType(type) {
  bgType = type;
  ['solid', 'gradient', 'photo'].forEach(t => {
    const btn = document.getElementById('btn-' + t);
    btn.style.borderColor = t === type ? '#a855f7' : 'rgba(255,255,255,0.1)';
    btn.style.background = t === type ? 'rgba(168,85,247,0.2)' : 'transparent';
  });
  const container = document.getElementById('bg-controls-container');
  if (type === 'solid') container.innerHTML = `<input type="color" value="${bgColor1}" onchange="bgColor1=this.value;updatePreview();" style="width:100%;height:40px;border-radius:8px;cursor:pointer;">`;
  else if (type === 'gradient') container.innerHTML = `<div style="display:flex;gap:8px;"><input type="color" value="${bgColor1}" onchange="bgColor1=this.value;updatePreview();" style="flex:1;height:40px;"><input type="color" value="${bgColor2}" onchange="bgColor2=this.value;updatePreview();" style="flex:1;height:40px;"></div>`;
  else container.innerHTML = `<label class="btn-secondary" style="display:block;text-align:center;cursor:pointer;">Choose Photo<input type="file" accept="image/*" style="display:none;" onchange="handlePostPhotoUpload(event)"></label>`;
  updatePreview();
}

async function handlePostPhotoUpload(e) {
  const file = e.target.files[0];
  if (!file) return;
  toast('Compressing...');
  bgPhotoDataUrl = await compressImageFile(file, 240, 0.5);
  updatePreview();
}

async function publishContent() {
  const bgStyle = bgType === 'photo' && bgPhotoDataUrl ? `url('${bgPhotoDataUrl}')` : (bgType === 'solid' ? bgColor1 : `linear-gradient(135deg, ${bgColor1}, ${bgColor2})`);
  if (createMode === 'post') {
    const r = await window.dataSdk.create({ type: 'post', post_owner: currentUser.username, post_texts: JSON.stringify(textItems), post_background: bgStyle, post_date: new Date().toISOString() });
    if (r.isOk) { toast('Shared post! ✨'); showScreen('screen-feed'); }
  } else {
    const r = await window.dataSdk.create({ type: 'story', story_owner: currentUser.username, story_texts: JSON.stringify(textItems), story_background: bgStyle, story_date: new Date().toISOString() });
    if (r.isOk) { toast('Story posted! 📸'); showScreen('screen-feed'); }
  }
}

// ===== STORIES FEED =====
function renderStories() {
  const friendNames = getMyFriendUsernames();
  const validStories = getStories().filter(s => s.story_owner === currentUser.username || friendNames.includes(s.story_owner));
  document.getElementById('stories-list').innerHTML = validStories.map(s => `
    <div class="story-bubble" onclick="viewStory('${s.__backendId}')">
      <div class="story-ring"><div class="story-inner">${avatarHTML(s.story_owner, 52)}</div></div>
      <span style="font-size:11px;color:rgba(255,255,255,0.8);">@${s.story_owner}</span>
    </div>
  `).join('');
}

function viewStory(id) {
  const s = allData.find(d => d.__backendId === id);
  if (!s) return;
  let textHtml = '';
  try {
    textHtml = JSON.parse(s.story_texts || '[]').map(i => `<div style="position:absolute;left:${i.x}%;top:${i.y}%;color:${i.color};font-size:${i.size}px;font-weight:700;text-align:center;">${escapeHtml(i.text)}</div>`).join('');
  } catch(e) {}
  const viewer = document.getElementById('story-viewer-content');
  viewer.style.background = s.story_background;
  viewer.innerHTML = `<div style="display:flex;align-items:center;gap:10px;">${avatarHTML(s.story_owner, 36)}<span style="font-weight:600;color:white;">@${s.story_owner}</span></div>${textHtml}<div></div>`;
  showScreen('screen-view-story');
}

function renderFeed() {
  if (!currentUser) return;
  const friendNames = getMyFriendUsernames();
  const posts = getPosts().filter(p => p.post_owner === currentUser.username || friendNames.includes(p.post_owner)).sort((a,b) => new Date(b.post_date) - new Date(a.post_date));
  
  document.getElementById('feed-empty').style.display = posts.length ? 'none' : 'block';
  document.getElementById('feed-posts').innerHTML = posts.map(p => {
    let postContent = '';
    try {
      postContent = JSON.parse(p.post_texts || '[]').map(i => `<div style="position:absolute;left:${i.x}%;top:${i.y}%;transform:translate(-50%,-50%);color:${i.color};font-size:${i.size}px;font-weight:700;text-shadow:0 2px 8px rgba(0,0,0,0.5);text-align:center;">${escapeHtml(i.text)}</div>`).join('');
    } catch(e) {}
    const ownerRec = getUserRecord(p.post_owner);
    return `
      <div class="post-card fade-in">
        <div style="display:flex;align-items:center;gap:10px;padding:12px 14px;">
          ${avatarHTML(p.post_owner, 36)}
          <div style="cursor:pointer;" onclick="viewUserProfile('${p.post_owner}')"><span style="font-weight:600;font-size:14px;">${escapeHtml(ownerRec?.display_name || p.post_owner)}</span><span style="font-size:11px;color:rgba(255,255,255,0.4);margin-left:6px;">@${p.post_owner}</span></div>
        </div>
        <div class="post-content-wrap" style="background:${p.post_background};background-size:cover;background-position:center;">${postContent}</div>
      </div>
    `;
  }).join('');
}

// ===== MESSAGES & GROUP CHATS =====
function renderChatsList() {
  if (!currentUser) return;
  const friends = getMyFriendUsernames();
  const groups = getGroups();
  const sidebar = document.getElementById('dm-chats-list');
  document.getElementById('dm-empty-state').style.display = (friends.length || groups.length) ? 'none' : 'block';
  
  let html = '';
  groups.forEach(g => {
    html += `
      <button onclick="openGroupChat('${g.__backendId}')" style="background:${currentChatId === g.__backendId ? 'rgba(168,85,247,0.2)' : 'transparent'};border:none;border-radius:10px;padding:12px;width:100%;cursor:pointer;display:flex;align-items:center;gap:12px;color:white;margin-bottom:4px;">
        <div class="avatar-circle" style="background:#6d28d9;"><i data-lucide="users" style="width:20px;height:20px;"></i></div>
        <div style="flex:1;text-align:left;"><div style="font-weight:600;font-size:14px;">${escapeHtml(g.group_name)}</div><span style="font-size:11px;color:#a855f7;">Group Chat</span></div>
      </button>
    `;
  });
  friends.forEach(f => {
    const u = getUserRecord(f);
    html += `
      <button onclick="openDmChat('${f}')" style="background:${currentChatId === f ? 'rgba(168,85,247,0.2)' : 'transparent'};border:none;border-radius:10px;padding:12px;width:100%;cursor:pointer;display:flex;align-items:center;gap:12px;color:white;margin-bottom:4px;">
        ${avatarHTML(f, 36)}
        <div style="flex:1;text-align:left;"><div style="font-weight:600;font-size:14px;">${escapeHtml(u?.display_name || f)}</div><span style="font-size:11px;color:rgba(255,255,255,0.4);">@${f}</span></div>
      </button>
    `;
  });
  sidebar.innerHTML = html;
  lucide.createIcons();
}

function openDmChat(uname) {
  currentChatId = uname;
  const u = getUserRecord(uname);
  document.getElementById('dm-header-name').textContent = u?.display_name || uname;
  document.getElementById('dm-chat-area').style.display = 'flex';
  renderMessages();
  renderChatsList();
}

function openGroupChat(gid) {
  currentChatId = gid;
  const g = allData.find(d => d.__backendId === gid);
  document.getElementById('dm-header-name').textContent = g.group_name;
  document.getElementById('dm-chat-area').style.display = 'flex';
  renderMessages();
  renderChatsList();
}

function closeDmChat() { currentChatId = null; document.getElementById('dm-chat-area').style.display = 'none'; }

function renderMessages() {
  const container = document.getElementById('dm-messages-container');
  let msgs = [];
  if (currentChatId.startsWith('id_')) {
    msgs = allData.filter(d => d.type === 'group_message' && d.group_id === currentChatId);
  } else {
    msgs = allData.filter(d => d.type === 'dm' && ((d.dm_from === currentUser.username && d.dm_to === currentChatId) || (d.dm_from === currentChatId && d.dm_to === currentUser.username)));
  }
  msgs.sort((a,b) => new Date(a.date) - new Date(b.date));

  container.innerHTML = msgs.map(m => {
    const isOwn = m.sender === currentUser.username || m.dm_from === currentUser.username;
    const senderName = isOwn ? 'You' : (m.sender || m.dm_from);
    return `
      <div style="display:flex;flex-direction:column;align-items:${isOwn ? 'flex-end' : 'flex-start'};">
        <span style="font-size:11px;color:rgba(255,255,255,0.4);margin-bottom:2px;">${senderName}</span>
        <div style="background:${isOwn ? 'linear-gradient(135deg, #a855f7, #6d28d9)' : 'rgba(255,255,255,0.08)'};padding:10px 14px;border-radius:12px;max-width:75%;color:white;font-size:14px;">${escapeHtml(m.text || m.dm_texts)}</div>
      </div>
    `;
  }).join('');
  container.scrollTop = container.scrollHeight;
}

async function sendDmMessage() {
  const text = document.getElementById('dm-input').value.trim();
  if (!text || !currentChatId) return;
  if (currentChatId.startsWith('id_')) {
    await window.dataSdk.create({ type: 'group_message', group_id: currentChatId, sender: currentUser.username, text, date: new Date().toISOString() });
  } else {
    await window.dataSdk.create({ type: 'dm', dm_from: currentUser.username, dm_to: currentChatId, dm_texts: text, dm_date: new Date().toISOString(), dm_type: 'text' });
  }
  document.getElementById('dm-input').value = '';
  renderMessages();
}

function openGroupModal() {
  const friends = getMyFriendUsernames();
  document.getElementById('group-friends-checkboxes').innerHTML = friends.map(f => `<label style="display:flex;align-items:center;gap:8px;font-size:14px;"><input type="checkbox" value="${f}" class="group-friend-chk" style="accent-color:#a855f7;"> @${f}</label>`).join('');
  document.getElementById('group-modal').classList.add('active');
}
function closeGroupModal() { document.getElementById('group-modal').classList.remove('active'); }

async function createGroupChat() {
  const name = document.getElementById('group-name-input').value.trim();
  const checked = Array.from(document.querySelectorAll('.group-friend-chk:checked')).map(c => c.value);
  if (!name || !checked.length) { toast('Name and 1+ friend required', true); return; }
  checked.push(currentUser.username);
  const r = await window.dataSdk.create({ type: 'group', group_name: name, group_members: checked });
  if (r.isOk) { toast('Group created!'); closeGroupModal(); renderChatsList(); }
}

// ===== DISCORD VOICE CALL SYSTEM =====
let discordTimer = null, discordSeconds = 0;
function startCurrentChatCall() {
  if (!currentChatId) return;
  let members = [currentUser.username];
  let title = 'Voice Channel';
  if (currentChatId.startsWith('id_')) {
    const g = allData.find(d => d.__backendId === currentChatId);
    members = g.group_members;
    title = g.group_name;
  } else {
    members.push(currentChatId);
    title = '@' + currentChatId;
  }
  document.getElementById('discord-call-title').textContent = title;
  document.getElementById('discord-grid').innerHTML = members.map(m => `
    <div class="discord-user-tile speaking">
      ${avatarHTML(m, 56)}
      <span style="font-weight:600;margin-top:8px;font-size:14px;">${m}</span>
    </div>
  `).join('');
  document.getElementById('discord-call-screen').classList.add('active');
  discordSeconds = 0;
  clearInterval(discordTimer);
  discordTimer = setInterval(() => {
    discordSeconds++;
    document.getElementById('discord-call-timer').textContent = `${String(Math.floor(discordSeconds/60)).padStart(2,'0')}:${String(discordSeconds%60).padStart(2,'0')}`;
  }, 1000);
}

function toggleDiscordMute() { toast('Toggled mic'); }
function toggleDiscordDeafen() { toast('Toggled deafen'); }
function endDiscordCall() {
  clearInterval(discordTimer);
  document.getElementById('discord-call-screen').classList.remove('active');
}

lucide.createIcons();
  </script>
</body>
</html>