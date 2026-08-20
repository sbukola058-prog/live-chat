<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Classroom Forum</title>

  <meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">
  <meta http-equiv="Pragma" content="no-cache">
  <meta http-equiv="Expires" content="0">

  <meta name="theme-color" content="#0b141a">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/3048/3048122.png">

  <script>
    const manifest = {
      "name": "Classroom Forum",
      "short_name": "Classroom",
      "start_url": ".",
      "display": "standalone",
      "background_color": "#0b141a",
      "theme_color": "#0b141a",
      "icons": [{ "src": "https://cdn-icons-png.flaticon.com/512/3048/3048122.png", "sizes": "512x512", "type": "image/png" }]
    };
    const link = document.createElement('link');
    link.rel = 'manifest';
    link.href = URL.createObjectURL(new Blob([JSON.stringify(manifest)], {type: 'application/json'}));
    document.head.appendChild(link);
  </script>

  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body { touch-action: manipulation; background-color: #0b141a; color: #e9edef; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }
    .whatsapp-bg {
      background-color: #0b141a;
      background-image: radial-gradient(#202c33 1px, transparent 0);
      background-size: 24px 24px;
    }
    .avatar-lecturer { background: linear-gradient(135deg, #FFD700, #FF8C00); padding: 2px; }
    .avatar-student { background: linear-gradient(135deg, #00a884, #25F4EE); padding: 1.5px; }
    .bubble-me { background-color: #005c4b; color: #e9edef; border-radius: 12px 0px 12px 12px; }
    .bubble-other { background-color: #202c33; color: #e9edef; border-radius: 0px 12px 12px 12px; }
    .bubble-lecturer { background-color: #1f2c34; border: 1px solid #FFD700; color: #fff; border-radius: 0px 12px 12px 12px; }
  </style>
</head>
<body class="fixed inset-0 w-full h-full bg-[#0b141a] text-white overflow-hidden">

  <!-- AUTH SCREEN -->
  <div id="auth-container" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-[#0b141a]">
    <div class="w-full max-w-sm p-6 bg-[#111b21] border border-[#222d34] rounded-3xl shadow-2xl">
      <div class="text-center mb-6">
        <div class="w-14 h-14 bg-[#00a884]/20 border border-[#00a884] rounded-2xl mx-auto flex items-center justify-center text-2xl mb-3">🎓</div>
        <h1 class="text-2xl font-black tracking-wider text-[#00a884]">CLASSROOM FORUM</h1>
        <p class="text-xs text-zinc-400 mt-1">Strengthen Your Mind Together</p>
      </div>

      <div id="login-error" class="hidden mb-4 p-3 bg-red-950/80 border border-red-500/80 text-red-300 text-xs rounded-2xl text-center font-semibold"></div>

      <input id="email" type="text" placeholder="Username (e.g. student01 or lecturer)" class="w-full p-3.5 bg-[#202c33] border border-[#2a3942] text-white rounded-2xl mb-3 focus:outline-none focus:border-[#00a884] text-sm">
      <input id="password" type="password" placeholder="Password" class="w-full p-3.5 bg-[#202c33] border border-[#2a3942] text-white rounded-2xl mb-5 focus:outline-none focus:border-[#00a884] text-sm">
      <button onclick="login()" class="w-full bg-[#00a884] hover:bg-[#008f6f] text-black p-3.5 rounded-2xl font-bold text-sm active:scale-95 transition">Enter Classroom</button>
      
      <button id="pwa-install-btn" onclick="installPWA()" class="w-full mt-3 bg-[#202c33] border border-[#00a884]/40 text-[#00a884] p-3 rounded-2xl font-bold text-xs hover:bg-[#202c33]/80 active:scale-95 transition flex items-center justify-center gap-2">
        📲 Install App on Phone
      </button>
    </div>
  </div>

  <!-- MAIN GROUP DASHBOARD -->
  <div id="group-dashboard" class="hidden fixed inset-0 w-full max-w-md mx-auto bg-[#0b141a] flex-col z-40">
    
    <!-- HEADER -->
    <div class="p-3 bg-[#111b21] border-b border-[#222d34] flex items-center justify-between shrink-0 gap-2">
      <div class="flex items-center gap-2.5 min-w-0 flex-1">
        <div id="user-avatar-wrapper" class="avatar-student rounded-full shrink-0">
          <div id="user-avatar-display" class="w-9 h-9 bg-[#202c33] rounded-full flex items-center justify-center text-xs font-bold text-white uppercase">US</div>
        </div>
        <div class="min-w-0 flex-1">
          <div class="flex items-center gap-1.5 flex-wrap">
            <h2 id="user-handle-display" class="font-bold text-xs text-white truncate max-w-[120px]">@username</h2>
            <span id="user-role-badge" class="text-[8px] bg-[#00a884]/20 text-[#00a884] px-1.5 py-0.5 rounded-full font-bold border border-[#00a884]/30 shrink-0">STUDENT</span>
          </div>
          <p class="text-[9px] text-[#00a884] flex items-center gap-1 mt-0.5">
            <span class="w-1.5 h-1.5 rounded-full bg-[#00a884] animate-pulse"></span> Classroom Group Active
          </p>
        </div>
      </div>
      <div class="flex items-center gap-1.5 shrink-0">
        <button onclick="installPWA()" class="text-[11px] bg-[#202c33] text-[#00a884] px-2 py-1 rounded-full border border-[#00a884]/40 font-semibold">📲 Install</button>
        <button onclick="logout()" class="text-[11px] bg-[#202c33] text-zinc-300 px-2.5 py-1 rounded-full border border-[#222d34] font-semibold">Exit</button>
      </div>
    </div>

    <!-- LIVE CHAT CONTAINER -->
    <div id="group-messages-box" class="flex-1 min-h-0 p-3.5 overflow-y-auto space-y-3 whatsapp-bg"></div>

    <!-- RESTRICTION LOCK BANNER (If Student is Muted) -->
    <div id="restriction-banner" class="hidden bg-amber-950/90 border-t border-amber-500/50 p-2.5 text-center shrink-0">
      <p id="restriction-text" class="text-xs text-amber-200 font-semibold">🔒 You are temporarily muted by the lecturer. You can still react to messages with emojis!</p>
    </div>

    <!-- INPUT BAR -->
    <div id="input-bar-container" class="p-2.5 bg-[#111b21] border-t border-[#222d34] flex flex-col gap-1 shrink-0">
      <div class="flex items-center gap-2 bg-[#202c33] border border-[#2a3942] rounded-full px-3 py-1.5">
        <label class="cursor-pointer text-zinc-400 hover:text-white shrink-0 text-lg" title="Attach image, document, link up to 50MB">
          📎
          <input type="file" id="group-file-input" class="hidden" onchange="handleFileSelect(this)">
        </label>
        <textarea id="group-message-input" rows="1" placeholder="Type message to classroom..." class="flex-1 bg-transparent text-white focus:outline-none resize-none max-h-20 text-sm px-1 py-1" onkeydown="handleKeyInput(event)"></textarea>
        <button onclick="sendGroupMessage()" class="text-[#00a884] font-bold text-sm px-2 shrink-0 hover:opacity-80">Send</button>
      </div>
      <div id="group-file-name" class="text-[10px] text-[#00a884] font-semibold hidden truncate px-3"></div>
    </div>
  </div>

  <!-- LECTURER RESTRICT / MUTE MODAL -->
  <div id="lecturer-mute-modal" class="hidden fixed inset-0 z-50 bg-black/80 flex items-center justify-center p-4">
    <div class="w-full max-w-xs bg-[#111b21] border border-[#222d34] p-5 rounded-3xl shadow-2xl text-center">
      <h3 class="text-sm font-bold text-amber-400 mb-1">🎓 Restrict Student Messaging</h3>
      <p id="target-student-label" class="text-xs text-zinc-300 mb-4">Mute @student01</p>

      <div class="grid grid-cols-2 gap-2 mb-4">
        <button onclick="applyRestriction(5)" class="p-2.5 bg-[#202c33] hover:bg-[#2a3942] text-xs font-bold rounded-xl border border-amber-500/30 text-amber-300">⏱️ 5 Minutes</button>
        <button onclick="applyRestriction(10)" class="p-2.5 bg-[#202c33] hover:bg-[#2a3942] text-xs font-bold rounded-xl border border-amber-500/30 text-amber-300">⏱️ 10 Minutes</button>
        <button onclick="applyRestriction(15)" class="p-2.5 bg-[#202c33] hover:bg-[#2a3942] text-xs font-bold rounded-xl border border-amber-500/30 text-amber-300">⏱️ 15 Minutes</button>
        <button onclick="applyRestriction(30)" class="p-2.5 bg-[#202c33] hover:bg-[#2a3942] text-xs font-bold rounded-xl border border-amber-500/30 text-amber-300">⏱️ 30 Minutes</button>
      </div>

      <div class="flex gap-2">
        <button onclick="liftRestriction()" class="flex-1 p-2 bg-emerald-950 border border-emerald-500/50 text-emerald-300 text-xs font-bold rounded-xl">Unmute Student</button>
        <button onclick="closeMuteModal()" class="p-2 bg-[#202c33] text-zinc-400 text-xs font-bold rounded-xl">Cancel</button>
      </div>
    </div>
  </div>

  <script>
    const SUPABASE_URL = "https://asovhkrwuirhzubvzezb.supabase.co";
    const SUPABASE_ANON_KEY = "sb_publishable_yoWyQctUADKZGOyYjYYHcQ_N1rmJa-9";
    const supabaseClient = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

    const SESSION_TIMEOUT_MS = 20 * 60 * 1000;
    let currentUser = null;
    let userProfile = null;
    let renderedMessageIds = new Set();
    let allLoadedMessages = [];
    let messageReactionsMap = {}; 
    let currentMuteUntil = null;
    let activeMuteCheckInterval = null;
    let selectedTargetStudent = null;
    let deferredPrompt = null;

    window.addEventListener('beforeinstallprompt', (e) => { e.preventDefault(); deferredPrompt = e; });

    async function installPWA() {
      const userAgent = navigator.userAgent || '';
      const isIOS = /iPad|iPhone|iPod/.test(userAgent) || (navigator.platform === 'MacIntel' && navigator.maxTouchPoints > 1);
      const isChromeIOS = /CriOS/.test(userAgent);

      if (isIOS) {
        if (isChromeIOS) {
          alert("🍎 iPhone Users (Chrome Detected):\n\n1. Copy this website URL.\n2. Open Safari on your iPhone.\n3. Paste the URL into Safari.\n4. Tap Share (⎋) → 'Add to Home Screen' (+).");
        } else {
          alert("🍎 iPhone Installation Steps (Safari):\n\n1. Tap the Share button (⎋) at the bottom.\n2. Scroll down and tap 'Add to Home Screen' (+).\n3. Tap 'Add' in top-right.");
        }
        return;
      }

      if (deferredPrompt) {
        deferredPrompt.prompt();
        const { outcome } = await deferredPrompt.userChoice;
        if (outcome === 'accepted') deferredPrompt = null;
      } else {
        alert("📲 To install on your phone:\n\n• Android: Tap 3 dots menu → 'Add to Home Screen'.\n• iPhone: Open in Safari → Tap Share (⎋) → 'Add to Home Screen'.");
      }
    }

    function updateActivityTimestamp() {
      if (currentUser) localStorage.setItem('classroom_last_activity', Date.now().toString());
    }
    ['mousemove', 'keydown', 'touchstart', 'click', 'scroll'].forEach(evt => window.addEventListener(evt, updateActivityTimestamp, { passive: true }));

    window.onload = async () => {
      const { data: { session } } = await supabaseClient.auth.getSession();
      if (session) {
        const lastAct = parseInt(localStorage.getItem('classroom_last_activity') || '0');
        if (lastAct > 0 && (Date.now() - lastAct) >= SESSION_TIMEOUT_MS) {
          showLoginError("⏰ Session expired due to inactivity.");
          logout();
          return;
        }
        handleUserSession(session.user);
      }
    };

    function showLoginError(msg) {
      const errDiv = document.getElementById('login-error');
      if (errDiv) {
        errDiv.innerText = msg;
        errDiv.classList.remove('hidden');
      }
    }

    async function login() {
      showLoginError("");
      let email = document.getElementById('email').value.trim();
      const password = document.getElementById('password').value;

      if (!email || !password) return showLoginError("Please enter username and password");
      if (!email.includes('@')) email = `${email}@livechat.local`;

      const { data, error } = await supabaseClient.auth.signInWithPassword({ email, password });
      if (error) return showLoginError("Login failed: " + error.message);

      updateActivityTimestamp();
      handleUserSession(data.user);
    }

    async function logout() {
      localStorage.removeItem('classroom_last_activity');
      await supabaseClient.auth.signOut();
      location.reload();
    }

    async function handleUserSession(user) {
      currentUser = user;
      document.getElementById('auth-container').classList.add('hidden');

      let { data: profile } = await supabaseClient.from('profiles').select('*').eq('id', user.id).maybeSingle();
      
      const rawHandle = user.email ? user.email.split('@')[0] : 'user';
      const userRole = profile?.role ? profile.role.toLowerCase() : (rawHandle.toLowerCase().includes('lecturer') ? 'lecturer' : 'student');
      const cleanName = profile?.full_name || rawHandle;

      userProfile = { id: user.id, email: user.email, role: userRole, name: cleanName };

      document.getElementById('user-handle-display').innerText = `@${cleanName}`;
      document.getElementById('user-avatar-display').innerText = cleanName.substring(0, 2).toUpperCase();

      const badge = document.getElementById('user-role-badge');
      const wrapper = document.getElementById('user-avatar-wrapper');

      if (userRole === 'lecturer') {
        badge.innerText = "🎓 LECTURER";
        badge.className = "text-[8px] bg-amber-500/20 text-amber-400 px-1.5 py-0.5 rounded-full font-bold border border-amber-500/40 shrink-0";
        wrapper.className = "avatar-lecturer rounded-full shrink-0";
      } else {
        badge.innerText = "⚡ STUDENT";
        badge.className = "text-[8px] bg-[#00a884]/20 text-[#00a884] px-1.5 py-0.5 rounded-full font-bold border border-[#00a884]/30 shrink-0";
        wrapper.className = "avatar-student rounded-full shrink-0";
      }

      document.getElementById('group-dashboard').classList.remove('hidden');
      document.getElementById('group-dashboard').classList.add('flex');

      checkStudentMuteStatus();
      loadAllReactions();
      loadAllGroupMessages();
      subscribeToRealtimeChannels();
    }

    // CHECK IF CURRENT STUDENT IS RESTRICTED BY LECTURER
    async function checkStudentMuteStatus() {
      if (userProfile.role === 'lecturer') return;

      const { data: restriction } = await supabaseClient
        .from('student_restrictions')
        .select('*')
        .eq('student_id', currentUser.id)
        .gte('muted_until', new Date().toISOString())
        .order('created_at', { ascending: false })
        .maybeSingle();

      if (restriction) {
        currentMuteUntil = new Date(restriction.muted_until).getTime();
        startMuteCountdown();
      } else {
        liftMuteUI();
      }
    }

    function startMuteCountdown() {
      const banner = document.getElementById('restriction-banner');
      const inputBar = document.getElementById('input-bar-container');
      const bannerText = document.getElementById('restriction-text');

      banner.classList.remove('hidden');
      inputBar.classList.add('opacity-40', 'pointer-events-none');

      clearInterval(activeMuteCheckInterval);
      activeMuteCheckInterval = setInterval(() => {
        const remainingSec = Math.max(0, Math.floor((currentMuteUntil - Date.now()) / 1000));
        if (remainingSec <= 0) {
          clearInterval(activeMuteCheckInterval);
          liftMuteUI();
        } else {
          const mins = Math.floor(remainingSec / 60);
          const secs = remainingSec % 60;
          bannerText.innerText = `🔒 Lecturer restricted your messaging for ${mins}m ${secs}s. Emoji reactions allowed!`;
        }
      }, 1000);
    }

    function liftMuteUI() {
      currentMuteUntil = null;
      clearInterval(activeMuteCheckInterval);
      document.getElementById('restriction-banner').classList.add('hidden');
      document.getElementById('input-bar-container').classList.remove('opacity-40', 'pointer-events-none');
    }

    // LECTURER MUTE ACTION MODAL
    function openMuteModal(studentId, studentName) {
      if (userProfile.role !== 'lecturer') return;
      selectedTargetStudent = { id: studentId, name: studentName };
      document.getElementById('target-student-label').innerText = `Restrict @${studentName}`;
      document.getElementById('lecturer-mute-modal').classList.remove('hidden');
    }

    function closeMuteModal() {
      document.getElementById('lecturer-mute-modal').classList.add('hidden');
      selectedTargetStudent = null;
    }

    async function applyRestriction(minutes) {
      if (!selectedTargetStudent) return;
      const mutedUntil = new Date(Date.now() + minutes * 60 * 1000).toISOString();

      await supabaseClient.from('student_restrictions').insert({
        student_id: selectedTargetStudent.id,
        student_name: selectedTargetStudent.name,
        muted_until: mutedUntil,
        muted_by: currentUser.id
      });

      closeMuteModal();
    }

    async function liftRestriction() {
      if (!selectedTargetStudent) return;

      await supabaseClient.from('student_restrictions')
        .delete()
        .eq('student_id', selectedTargetStudent.id);

      closeMuteModal();
    }

    function handleKeyInput(event) {
      if ((event.key === 'Enter' || event.keyCode === 13) && !event.shiftKey) {
        event.preventDefault();
        sendGroupMessage();
      }
    }

    function handleFileSelect(input) {
      const label = document.getElementById('group-file-name');
      if (input.files.length > 0) {
        const file = input.files[0];
        if (file.size > 50 * 1024 * 1024) {
          alert("File exceeds 50MB limit!");
          input.value = '';
          label.classList.add('hidden');
          return;
        }
        label.innerText = `📎 Attached: ${file.name} (${(file.size / (1024 * 1024)).toFixed(1)}MB)`;
        label.classList.remove('hidden');
      } else {
        label.classList.add('hidden');
      }
    }

    async function uploadAttachment() {
      const fileInput = document.getElementById('group-file-input');
      if (!fileInput.files || fileInput.files.length === 0) return null;

      const file = fileInput.files[0];
      const fileExt = file.name.split('.').pop().toLowerCase();
      const fileName = `${Date.now()}_${Math.random().toString(36).substring(2, 7)}.${fileExt}`;
      const filePath = `${currentUser.id}/${fileName}`;

      const { data, error } = await supabaseClient.storage.from('chat-attachments').upload(filePath, file);
      if (error) {
        alert("Upload failed: " + error.message);
        return null;
      }

      const { data: publicUrlData } = supabaseClient.storage.from('chat-attachments').getPublicUrl(filePath);
      fileInput.value = '';
      document.getElementById('group-file-name').classList.add('hidden');

      const isImg = ['jpg', 'jpeg', 'png', 'gif', 'webp'].includes(fileExt);
      return { url: publicUrlData.publicUrl, type: isImg ? 'image' : 'file', name: file.name };
    }

    function formatMessageText(text) {
      if (!text) return '';
      let safeText = text.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
      const urlRegex = /(https?:\/\/[^\s]+)/g;
      safeText = safeText.replace(urlRegex, (url) => `<a href="${url}" target="_blank" rel="noopener noreferrer" class="underline text-[#25F4EE] font-semibold break-all">${url}</a>`);
      return safeText.replace(/\n/g, '<br>');
    }

    function getUserSpeedBadge(senderId) {
      const userMsgs = allLoadedMessages.filter(m => m.sender_id === senderId);
      if (userMsgs.length < 2) return { label: '🌟 New Member', class: 'bg-[#202c33] text-zinc-300 border-[#2a3942]' };

      let totalGaps = 0, gapCount = 0;
      for (let i = 1; i < userMsgs.length; i++) {
        const diffSec = (new Date(userMsgs[i].created_at) - new Date(userMsgs[i - 1].created_at)) / 1000;
        if (diffSec > 0 && diffSec < 300) { totalGaps += diffSec; gapCount++; }
      }

      if (gapCount === 0) return { label: '⚡ Lightning Fast', class: 'bg-emerald-500/20 text-emerald-400 border-emerald-500/40' };
      const avgSec = Math.round(totalGaps / gapCount);

      if (avgSec <= 15) return { label: `⚡ ${avgSec}s • Lightning`, class: 'bg-emerald-500/20 text-emerald-400 border-emerald-500/40' };
      if (avgSec <= 45) return { label: `🔥 ${avgSec}s • Fast`, class: 'bg-amber-500/20 text-amber-400 border-amber-500/40' };
      return { label: `💬 ${avgSec}s • Active`, class: 'bg-cyan-500/20 text-cyan-400 border-cyan-500/40' };
    }

    // INSTANT SEND
    async function sendGroupMessage() {
      if (currentMuteUntil && currentMuteUntil > Date.now()) {
        alert("🔒 You are currently restricted by the lecturer from typing messages.");
        return;
      }

      const input = document.getElementById('group-message-input');
      let content = input.value.trim();
      input.value = '';

      const attachment = await uploadAttachment();
      if (!content && !attachment) return;

      updateActivityTimestamp();

      const tempId = `temp_${Date.now()}`;
      const optimisticMsg = {
        id: tempId,
        sender_id: currentUser.id,
        sender_name: userProfile.name,
        sender_role: userProfile.role,
        content: content,
        attachment_url: attachment?.url || null,
        attachment_type: attachment?.type || null,
        attachment_name: attachment?.name || null,
        created_at: new Date().toISOString()
      };

      allLoadedMessages.push(optimisticMsg);
      appendSingleMessage(optimisticMsg);

      const { data, error } = await supabaseClient.from('messages').insert({
        student_id: currentUser.id,
        sender_id: currentUser.id,
        sender_name: userProfile.name,
        sender_role: userProfile.role,
        content: content,
        attachment_url: attachment?.url || null,
        attachment_type: attachment?.type || null,
        attachment_name: attachment?.name || null
      }).select().single();

      if (error) {
        const tempElement = document.getElementById(`msg-${tempId}`);
        if (tempElement) tempElement.classList.add('opacity-50', 'border-red-500');
      } else if (data) {
        const tempElement = document.getElementById(`msg-${tempId}`);
        if (tempElement) tempElement.id = `msg-${data.id}`;
        renderedMessageIds.delete(tempId);
        renderedMessageIds.add(data.id);
      }
    }

    async function loadAllReactions() {
      const { data: rxns } = await supabaseClient.from('message_reactions').select('*');
      messageReactionsMap = {};
      if (rxns) {
        rxns.forEach(r => {
          if (!messageReactionsMap[r.message_id]) messageReactionsMap[r.message_id] = [];
          messageReactionsMap[r.message_id].push(r);
        });
      }
    }

    async function loadAllGroupMessages() {
      const { data: messages } = await supabaseClient
        .from('messages')
        .select('*')
        .order('created_at', { ascending: true })
        .limit(150);

      const box = document.getElementById('group-messages-box');
      box.innerHTML = '';
      renderedMessageIds.clear();
      allLoadedMessages = messages || [];

      if (messages) messages.forEach(msg => appendSingleMessage(msg));
    }

    // EMOJI REACTION TOGGLE
    async function toggleEmojiReaction(msgId, emoji) {
      updateActivityTimestamp();
      const existing = (messageReactionsMap[msgId] || []).find(r => r.user_id === currentUser.id && r.emoji === emoji);

      if (existing) {
        await supabaseClient.from('message_reactions').delete().eq('id', existing.id);
      } else {
        await supabaseClient.from('message_reactions').insert({
          message_id: msgId,
          user_id: currentUser.id,
          user_name: userProfile.name,
          emoji: emoji
        });
      }
    }

    function renderReactionTray(msgId) {
      const rxns = messageReactionsMap[msgId] || [];
      if (rxns.length === 0) return '';

      const counts = {};
      rxns.forEach(r => counts[r.emoji] = (counts[r.emoji] || 0) + 1);

      let html = '<div class="flex items-center gap-1 mt-1 flex-wrap">';
      Object.keys(counts).forEach(e => {
        html += `<span class="text-[10px] bg-[#111b21] border border-[#2a3942] px-1.5 py-0.5 rounded-full flex items-center gap-1">${e} <span class="font-bold opacity-80">${counts[e]}</span></span>`;
      });
      html += '</div>';
      return html;
    }

    function appendSingleMessage(msg) {
      if (renderedMessageIds.has(msg.id)) return;
      renderedMessageIds.add(msg.id);

      const box = document.getElementById('group-messages-box');
      const isMe = msg.sender_id === currentUser.id;
      const isLecturer = (msg.sender_role || '').toLowerCase() === 'lecturer';

      const msgDiv = document.createElement('div');
      msgDiv.id = `msg-${msg.id}`;
      msgDiv.className = `flex flex-col ${isMe ? 'items-end' : 'items-start'} mb-3 group`;

      const timeStr = new Date(msg.created_at).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
      let displayName = msg.sender_name || 'student';
      if (displayName.includes('@')) displayName = displayName.split('@')[0];

      const speedBadge = getUserSpeedBadge(msg.sender_id);

      // HEADER
      let senderHeader = '';
      if (!isMe) {
        const roleTag = isLecturer 
          ? '<span class="text-[8px] bg-amber-500/20 text-amber-300 px-1.5 py-0.2 rounded font-bold border border-amber-500/30">🎓 LECTURER</span>'
          : '<span class="text-[8px] bg-zinc-800 text-zinc-400 px-1.5 py-0.2 rounded font-bold">STUDENT</span>';

        const restrictBtn = (userProfile.role === 'lecturer' && !isLecturer)
          ? `<button onclick="openMuteModal('${msg.sender_id}', '${displayName}')" class="text-[9px] bg-red-950 text-red-300 px-1.5 py-0.2 rounded font-bold border border-red-500/40 hover:bg-red-900 ml-1">🔇 Restrict</button>`
          : '';

        senderHeader = `
          <div class="flex items-center gap-1.5 mb-1 px-1 flex-wrap">
            <span class="text-[11px] font-bold text-zinc-200">@${displayName}</span>
            ${roleTag}
            <span class="text-[8px] px-1.5 py-0.2 rounded font-semibold border ${speedBadge.class}">${speedBadge.label}</span>
            ${restrictBtn}
          </div>
        `;
      } else {
        senderHeader = `
          <div class="flex items-center gap-1.5 mb-1 px-1 justify-end">
            <span class="text-[8px] px-1.5 py-0.2 rounded font-semibold border ${speedBadge.class}">${speedBadge.label}</span>
            <span class="text-[11px] font-bold text-zinc-300">You (@${displayName})</span>
          </div>
        `;
      }

      // ATTACHMENT DISPLAY
      let attachmentHTML = '';
      if (msg.attachment_url) {
        if (msg.attachment_type === 'image') {
          attachmentHTML = `<a href="${msg.attachment_url}" target="_blank"><img src="${msg.attachment_url}" class="max-h-52 rounded-xl my-1.5 object-cover border border-[#2a3942]" /></a>`;
        } else {
          attachmentHTML = `<a href="${msg.attachment_url}" target="_blank" download class="flex items-center gap-2 p-2 bg-[#111b21] rounded-xl my-1.5 border border-[#2a3942] text-xs text-[#00a884] font-semibold truncate"><span class="text-base">📄</span> ${msg.attachment_name || 'Download File'}</a>`;
        }
      }

      let contentText = msg.deleted_for_everyone 
        ? '<i class="opacity-60 text-zinc-400">This message was deleted</i>' 
        : formatMessageText(msg.content);

      let bubbleStyle = 'bubble-other';
      if (isMe) bubbleStyle = 'bubble-me';
      else if (isLecturer) bubbleStyle = 'bubble-lecturer';

      msgDiv.innerHTML = `
        ${senderHeader}
        <div class="max-w-[85%] p-3 text-xs relative ${bubbleStyle} shadow-md">
          ${attachmentHTML}
          <p class="whitespace-pre-wrap leading-relaxed">${contentText}</p>
          
          <div id="rxns-${msg.id}">
            ${renderReactionTray(msg.id)}
          </div>

          <div class="flex items-center justify-between gap-2 mt-1 text-[9px] opacity-60 border-t border-white/10 pt-1">
            <!-- QUICK EMOJI PICKER -->
            <div class="flex items-center gap-1 text-sm">
              <button onclick="toggleEmojiReaction('${msg.id}', '👍')" class="hover:scale-125 transition">👍</button>
              <button onclick="toggleEmojiReaction('${msg.id}', '❤️')" class="hover:scale-125 transition">❤️</button>
              <button onclick="toggleEmojiReaction('${msg.id}', '😂')" class="hover:scale-125 transition">😂</button>
              <button onclick="toggleEmojiReaction('${msg.id}', '💡')" class="hover:scale-125 transition">💡</button>
              <button onclick="toggleEmojiReaction('${msg.id}', '🔥')" class="hover:scale-125 transition">🔥</button>
            </div>
            
            <div class="flex items-center gap-1">
              <span>${timeStr}</span>
              ${(isMe && !msg.deleted_for_everyone) ? `
                <button onclick="deleteMessage('${msg.id}')" class="hidden group-hover:inline underline text-red-300 hover:text-red-200 ml-1">Delete</button>
              ` : ''}
            </div>
          </div>
        </div>
      `;

      box.appendChild(msgDiv);
      box.scrollTop = box.scrollHeight;
    }

    async function deleteMessage(msgId) {
      await supabaseClient.from('messages').update({ deleted_for_everyone: true }).eq('id', msgId);
      const msgText = document.querySelector(`#msg-${msgId} p`);
      if (msgText) msgText.innerHTML = '<i class="opacity-60 text-zinc-400">This message was deleted</i>';
    }

    function subscribeToRealtimeChannels() {
      supabaseClient
        .channel('public_group_chat')
        .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, (payload) => {
          allLoadedMessages.push(payload.new);
          appendSingleMessage(payload.new);
        })
        .on('postgres_changes', { event: 'UPDATE', schema: 'public', table: 'messages' }, (payload) => {
          if (payload.new.deleted_for_everyone) {
            const msgText = document.querySelector(`#msg-${payload.new.id} p`);
            if (msgText) msgText.innerHTML = '<i class="opacity-60 text-zinc-400">This message was deleted</i>';
          }
        })
        .on('postgres_changes', { event: '*', schema: 'public', table: 'message_reactions' }, async () => {
          await loadAllReactions();
          allLoadedMessages.forEach(m => {
            const el = document.getElementById(`rxns-${m.id}`);
            if (el) el.innerHTML = renderReactionTray(m.id);
          });
        })
        .on('postgres_changes', { event: '*', schema: 'public', table: 'student_restrictions' }, () => {
          checkStudentMuteStatus();
        })
        .subscribe();
    }
  </script>
</body>
</html>
