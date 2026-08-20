
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Classroom Live Chat</title>
  
  <!-- PWA & Mobile App Setup -->
  <meta name="theme-color" content="#000000">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="Classroom Feed">
  <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/3048/3048122.png">

  <!-- Dynamic PWA Manifest Injection -->
  <script>
    const manifest = {
      "name": "Classroom Live Chat",
      "short_name": "ClassFeed",
      "start_url": ".",
      "display": "standalone",
      "background_color": "#000000",
      "theme_color": "#000000",
      "icons": [
        {
          "src": "https://cdn-icons-png.flaticon.com/512/3048/3048122.png",
          "sizes": "512x512",
          "type": "image/png"
        }
      ]
    };
    const stringManifest = JSON.stringify(manifest);
    const blob = new Blob([stringManifest], {type: 'application/json'});
    const manifestURL = URL.createObjectURL(blob);
    const link = document.createElement('link');
    link.rel = 'manifest';
    link.href = manifestURL;
    document.head.appendChild(link);
  </script>

  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body {
      touch-action: manipulation;
    }
    .tiktok-avatar-ring {
      background: linear-gradient(45deg, #25F4EE, #FE2C55);
      padding: 2px;
    }
    .tiktok-card {
      background: #181818;
      border: 1px solid #282828;
    }
    .tiktok-card:active {
      border-color: #FE2C55;
      transform: scale(0.98);
    }
  </style>
</head>
<body class="bg-black text-white h-[100dvh] font-sans overflow-hidden select-none">

  <!-- LOGIN CONTAINER -->
  <div id="auth-container" class="max-w-md mx-[5%] mt-12 p-6 bg-zinc-900 border border-zinc-800 rounded-2xl shadow-2xl">
    <div class="text-center mb-6">
      <span class="text-3xl font-black tracking-wider text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 to-pink-500">CLASSROOM</span>
      <h2 class="text-lg font-bold text-zinc-300">Portal Login</h2>
    </div>
    <input id="email" type="text" placeholder="Username (e.g. student01 or lecturer)" class="w-full p-3.5 bg-zinc-800 border border-zinc-700 text-white rounded-xl mb-3 focus:outline-none focus:border-cyan-400" onkeydown="if(event.key==='Enter') login()">
    <input id="password" type="password" placeholder="Password" class="w-full p-3.5 bg-zinc-800 border border-zinc-700 text-white rounded-xl mb-5 focus:outline-none focus:border-cyan-400" onkeydown="if(event.key==='Enter') login()">
    <button onclick="login()" class="w-full bg-gradient-to-r from-cyan-500 to-pink-500 text-white p-3.5 rounded-xl font-bold hover:opacity-90 transition active:scale-95">Sign In</button>
  </div>

  <!-- 1. STUDENT DASHBOARD (MOBILE OPTIMIZED) -->
  <div id="student-dashboard" class="hidden flex flex-col h-[100dvh] w-full max-w-md mx-auto bg-gray-900 shadow-2xl">
    <!-- STUDENT HEADER -->
    <div class="p-3 border-b border-gray-800 flex justify-between items-center bg-gray-900 text-white">
      <div class="flex items-center gap-2">
        <div>
          <h2 id="student-username-display" class="font-bold text-sm">@student</h2>
          <span id="student-speed-badge" class="text-[10px] bg-blue-600 text-white px-2 py-0.5 rounded-full font-bold">⚡ Calculating...</span>
        </div>
      </div>

      <div class="flex items-center gap-2">
        <div class="flex flex-col items-end">
          <span class="text-[9px] text-gray-400 uppercase font-bold">Timer</span>
          <span id="student-timer-badge" class="text-xs bg-red-600 text-white px-2.5 py-0.5 rounded-full font-mono font-bold animate-pulse">⏱️ 02:00</span>
        </div>
        <button onclick="logout()" class="text-xs bg-zinc-800 hover:bg-zinc-700 text-zinc-300 px-2.5 py-1 rounded-lg border border-zinc-700">Exit</button>
      </div>
    </div>

    <div id="student-messages-box" class="flex-1 p-3 overflow-y-auto space-y-3 bg-black"></div>

    <!-- STUDENT INPUT -->
    <div class="p-3 border-t border-gray-800 bg-gray-900 flex flex-col gap-1.5">
      <div class="flex items-end gap-2">
        <label class="cursor-pointer bg-zinc-800 p-3 rounded-xl border border-zinc-700 flex items-center justify-center shrink-0" title="Attach file (Up to 50MB)">
          📎
          <input type="file" id="student-file-input" class="hidden" onchange="handleFileSelect(this, 'student-file-name')">
        </label>
        <textarea id="student-message-input" rows="1" placeholder="Message (Shift+Enter for paragraph)..." class="flex-1 p-2.5 bg-zinc-800 border border-zinc-700 text-white rounded-xl focus:outline-none focus:border-cyan-400 resize-none max-h-24 text-xs" onkeydown="handleKeyInput(event, 'student')"></textarea>
        <button onclick="sendStudentMessage()" class="bg-blue-600 text-white px-4 py-2.5 rounded-xl font-bold text-xs hover:bg-blue-700 shrink-0">Send</button>
      </div>
      <div id="student-file-name" class="text-[10px] text-cyan-400 font-semibold hidden truncate"></div>
    </div>
  </div>

  <!-- 2. LECTURER DASHBOARD (TIKTOK STYLE MOBILE RESPONSIVE) -->
  <div id="lecturer-dashboard" class="hidden h-[100dvh] w-full bg-black text-white flex flex-col">
    <!-- TOP BAR -->
    <div class="p-3 bg-zinc-900 border-b border-zinc-800 flex justify-between items-center px-4 shrink-0">
      <div class="flex items-center gap-2">
        <span class="text-lg font-black tracking-wider text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 to-pink-500">CLASSROOM FEED</span>
        <span class="text-[10px] bg-pink-600/30 text-pink-400 border border-pink-500/50 px-2 py-0.5 rounded-full font-bold">LECTURER</span>
      </div>
      <button onclick="logout()" class="text-xs bg-zinc-800 text-zinc-300 px-3 py-1 rounded-full border border-zinc-700">Logout</button>
    </div>

    <div class="flex-1 flex overflow-hidden relative">
      <!-- LEFT FEED (Mobile Full Width / Desktop 1/2 Width) -->
      <div id="lecturer-feed-panel" class="w-full md:w-1/2 p-3 overflow-y-auto space-y-3 border-r border-zinc-800 block md:block">
        <h3 class="text-[10px] font-bold text-zinc-400 uppercase tracking-widest mb-1">Student Profiles Feed</h3>
        <div id="tiktok-feed" class="space-y-2.5"></div>
      </div>

      <!-- RIGHT CHAT AREA (Mobile Full Screen Overlay / Desktop 1/2 Width) -->
      <div id="lecturer-chat-panel" class="hidden md:flex w-full md:w-1/2 flex-col bg-zinc-950 h-full absolute md:relative inset-0 z-10 md:z-auto">
        <!-- CHAT HEADER WITH BACK BUTTON FOR MOBILE -->
        <div id="lecturer-chat-header" class="p-3 border-b border-zinc-800 flex items-center justify-between bg-zinc-900 shrink-0">
          <div class="text-zinc-500 italic text-xs">Select a student from feed</div>
        </div>

        <div id="lecturer-messages-box" class="flex-1 p-3 overflow-y-auto space-y-3 bg-zinc-950"></div>

        <!-- LECTURER INPUT -->
        <div class="p-3 border-t border-zinc-800 flex flex-col gap-1.5 bg-zinc-900 shrink-0">
          <div class="flex items-end gap-2">
            <label class="cursor-pointer bg-zinc-800 p-3 rounded-xl border border-zinc-700 flex items-center justify-center shrink-0" title="Attach file (Up to 50MB)">
              📎
              <input type="file" id="lecturer-file-input" class="hidden" onchange="handleFileSelect(this, 'lecturer-file-name')">
            </label>
            <textarea id="lecturer-message-input" rows="1" placeholder="Reply to student..." class="flex-1 p-2.5 bg-zinc-800 border border-zinc-700 text-white rounded-xl focus:outline-none focus:border-cyan-400 resize-none max-h-24 text-xs" onkeydown="handleKeyInput(event, 'lecturer')"></textarea>
            <button onclick="sendLecturerMessage()" class="bg-gradient-to-r from-cyan-500 to-pink-500 text-white px-4 py-2.5 rounded-xl font-bold text-xs shrink-0">Reply</button>
          </div>
          <div id="lecturer-file-name" class="text-[10px] text-cyan-400 font-semibold hidden truncate"></div>
        </div>
      </div>
    </div>
  </div>

  <script>
    const SUPABASE_URL = "https://qkwlasexypebdvkbhgmd.supabase.co";
    const SUPABASE_ANON_KEY = "sb_publishable_lrONtIn4ve7PZvCWtNkuvg_KGwDlwvO";
    const supabaseClient = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

    let currentUser = null;
    let userProfile = null;
    let activeStudentId = null;
    let countdownTimer = null;
    let timeLeft = 120;

    // Register Service Worker for PWA installation
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        const swCode = `self.addEventListener('fetch', (e) => {});`;
        const blob = new Blob([swCode], { type: 'text/javascript' });
        const swUrl = URL.createObjectURL(blob);
        navigator.serviceWorker.register(swUrl).catch(() => {});
      });
    }

    window.onload = async () => {
      const { data: { session } } = await supabaseClient.auth.getSession();
      if (session) handleUserSession(session.user);
    };

    async function login() {
      let email = document.getElementById('email').value.trim();
      const password = document.getElementById('password').value;
      if (!email || !password) return alert("Please enter login details");
      if (!email.includes('@')) email = `${email}@livechat.local`;

      const { data, error } = await supabaseClient.auth.signInWithPassword({ email, password });
      if (error) return alert("Login failed: " + error.message);
      handleUserSession(data.user);
    }

    async function logout() {
      clearInterval(countdownTimer);
      await supabaseClient.auth.signOut();
      location.reload();
    }

    async function handleUserSession(user) {
      currentUser = user;
      document.getElementById('auth-container').classList.add('hidden');

      let { data: profile } = await supabaseClient.from('profiles').select('*').eq('id', user.id).maybeSingle();
      const userRole = profile?.role ? profile.role.toLowerCase() : 'student';
      userProfile = profile || { id: user.id, email: user.email, role: userRole };

      if (userRole === 'lecturer') {
        document.getElementById('lecturer-dashboard').classList.remove('hidden');
        loadTikTokStudentFeed();
      } else {
        document.getElementById('student-dashboard').classList.remove('hidden');
        activeStudentId = user.id;

        const cleanUsername = (profile?.full_name || user.email.split('@')[0]);
        document.getElementById('student-username-display').innerText = `@${cleanUsername}`;

        startCountdownTimer();
        loadStudentMessages();
      }
      subscribeToMessages();
    }

    /* --- SHIFT + ENTER PARAGRAPH HANDLER --- */
    function handleKeyInput(event, role) {
      if (event.key === 'Enter' && !event.shiftKey) {
        event.preventDefault();
        if (role === 'student') sendStudentMessage();
        else sendLecturerMessage();
      }
    }

    function handleFileSelect(input, labelId) {
      const label = document.getElementById(labelId);
      if (input.files.length > 0) {
        const file = input.files[0];
        if (file.size > 50 * 1024 * 1024) {
          alert("File exceeds maximum size limit of 50MB!");
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

    /* --- FILE UPLOAD TO SUPABASE STORAGE --- */
    async function uploadAttachment(fileInputId) {
      const fileInput = document.getElementById(fileInputId);
      if (!fileInput.files || fileInput.files.length === 0) return null;

      const file = fileInput.files[0];
      const fileExt = file.name.split('.').pop();
      const fileName = `${Date.now()}_${Math.random().toString(36).substring(2, 7)}.${fileExt}`;
      const filePath = `${currentUser.id}/${fileName}`;

      const { data, error } = await supabaseClient.storage
        .from('chat-attachments')
        .upload(filePath, file);

      if (error) {
        alert("File upload failed: " + error.message);
        return null;
      }

      const { data: publicUrlData } = supabaseClient.storage
        .from('chat-attachments')
        .getPublicUrl(filePath);

      fileInput.value = '';
      return { url: publicUrlData.publicUrl, name: file.name, type: file.type };
    }

    /* --- AUTO-LINKIFY & HTML FORMATTING --- */
    function formatMessageText(text) {
      if (!text) return '';
      let safeText = text.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
      const urlRegex = /(https?:\/\/[^\s]+)/g;
      safeText = safeText.replace(urlRegex, (url) => {
        return `<a href="${url}" target="_blank" rel="noopener noreferrer" class="underline text-cyan-300 font-semibold break-all">${url}</a>`;
      });
      return safeText.replace(/\n/g, '<br>');
    }

    /* --- TIMER & SPEED FUNCTIONS --- */
    function startCountdownTimer() {
      clearInterval(countdownTimer);
      timeLeft = 120;
      updateTimerDisplay();

      countdownTimer = setInterval(() => {
        timeLeft--;
        updateTimerDisplay();
        if (timeLeft <= 0) {
          clearInterval(countdownTimer);
          alert("⏰ TIME IS UP! You failed to reply with at least 10 characters within 2 minutes.");
          logout();
        }
      }, 1000);
    }

    function updateTimerDisplay() {
      const minutes = Math.floor(timeLeft / 60);
      const seconds = timeLeft % 60;
      const formatted = `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
      const badge = document.getElementById('student-timer-badge');
      if (badge) badge.innerText = `⏱️ ${formatted}`;
    }

    function calculateAvgResponseSpeed(messages, studentId) {
      let totalSpeedSec = 0;
      let count = 0;

      for (let i = 0; i < messages.length - 1; i++) {
        const currentMsg = messages[i];
        const nextMsg = messages[i + 1];

        if (currentMsg.sender_id !== studentId && nextMsg.sender_id === studentId) {
          const lecturerTime = new Date(currentMsg.created_at).getTime();
          const studentTime = new Date(nextMsg.created_at).getTime();
          const diffSec = (studentTime - lecturerTime) / 1000;

          if (diffSec >= 0) {
            totalSpeedSec += diffSec;
            count++;
          }
        }
      }

      if (count === 0) return { label: "⚡ No Replies", color: "bg-gray-600 text-white" };

      const avgSec = Math.round(totalSpeedSec / count);
      let label = `⚡ ${avgSec}s`;
      let color = "bg-emerald-500 text-white";

      if (avgSec > 60) color = "bg-red-500 text-white";
      else if (avgSec > 30) color = "bg-amber-500 text-white";

      return { label, color };
    }

    /* --- LECTURER DASHBOARD (MOBILE NAVIGATION SLIDER) --- */
    async function loadTikTokStudentFeed() {
      const { data: students } = await supabaseClient.from('profiles').select('*').eq('role', 'student');
      const feedContainer = document.getElementById('tiktok-feed');
      feedContainer.innerHTML = '';

      if (!students || students.length === 0) {
        feedContainer.innerHTML = '<p class="text-xs text-zinc-500">No student profiles found.</p>';
        return;
      }

      const studentFeedData = await Promise.all(students.map(async (student) => {
        const { data: msgs } = await supabaseClient
          .from('messages')
          .select('*')
          .eq('student_id', student.id)
          .order('created_at', { ascending: false });

        const latestMsg = msgs && msgs.length > 0 ? msgs[0] : null;
        const speedData = calculateAvgResponseSpeed(msgs ? msgs.reverse() : [], student.id);

        return {
          student,
          previewText: latestMsg ? latestMsg.content : "No messages yet",
          lastTimestamp: latestMsg ? new Date(latestMsg.created_at).getTime() : 0,
          speedData
        };
      }));

      studentFeedData.sort((a, b) => b.lastTimestamp - a.lastTimestamp);

      studentFeedData.forEach(({ student, previewText, speedData }) => {
        const studentName = student.full_name || student.email.split('@')[0];

        const card = document.createElement('div');
        card.className = `tiktok-card p-3 rounded-2xl flex items-center justify-between cursor-pointer transition ${activeStudentId === student.id ? 'border-cyan-400 bg-zinc-900' : ''}`;
        card.onclick = () => selectStudentThread(student);

        card.innerHTML = `
          <div class="flex items-center gap-2.5">
            <div class="tiktok-avatar-ring rounded-full shrink-0">
              <div class="w-10 h-10 bg-zinc-900 rounded-full flex items-center justify-center text-xs font-bold text-white uppercase border border-zinc-800">
                ${studentName.substring(0, 2)}
              </div>
            </div>
            <div class="overflow-hidden">
              <div class="font-bold text-xs text-zinc-100">@${studentName}</div>
              <p class="text-[11px] text-zinc-400 truncate max-w-[130px] sm:max-w-[180px]">${previewText}</p>
            </div>
          </div>
          <span class="text-[9px] px-2 py-0.5 rounded-full font-bold ${speedData.color} shrink-0">${speedData.label}</span>
        `;
        feedContainer.appendChild(card);
      });
    }

    function closeMobileChat() {
      document.getElementById('lecturer-chat-panel').classList.add('hidden');
    }

    async function selectStudentThread(student) {
      activeStudentId = student.id;
      const studentName = student.full_name || student.email.split('@')[0];

      // Show chat panel on mobile
      const chatPanel = document.getElementById('lecturer-chat-panel');
      chatPanel.classList.remove('hidden');

      const { data: messages } = await supabaseClient
        .from('messages')
        .select('*')
        .eq('student_id', student.id)
        .order('created_at', { ascending: true });

      const speedData = calculateAvgResponseSpeed(messages || [], student.id);

      document.getElementById('lecturer-chat-header').innerHTML = `
        <div class="flex items-center gap-2">
          <button onclick="closeMobileChat()" class="md:hidden text-lg bg-zinc-800 px-2.5 py-1 rounded-lg text-zinc-300">←</button>
          <div class="tiktok-avatar-ring rounded-full">
            <div class="w-7 h-7 bg-zinc-900 rounded-full flex items-center justify-center text-[10px] font-bold text-white uppercase">
              ${studentName.substring(0, 2)}
            </div>
          </div>
          <div>
            <h3 class="font-bold text-xs text-white">@${studentName}</h3>
            <p class="text-[9px] text-emerald-400 font-semibold">Active Session</p>
          </div>
        </div>
        <span class="text-[10px] px-2 py-0.5 rounded-full font-bold ${speedData.color}">${speedData.label}</span>
      `;

      renderMessages(messages || [], 'lecturer-messages-box');
      loadTikTokStudentFeed();
    }

    async function loadLecturerMessages() {
      if (!activeStudentId) return;

      const { data: messages } = await supabaseClient
        .from('messages')
        .select('*')
        .eq('student_id', activeStudentId)
        .order('created_at', { ascending: true });

      renderMessages(messages || [], 'lecturer-messages-box');
    }

    async function sendLecturerMessage() {
      const input = document.getElementById('lecturer-message-input');
      let content = input.value.trim();
      const fileData = await uploadAttachment('lecturer-file-input');

      if (fileData) {
        content = content ? `${content}\n\n📎 Attachment: ${fileData.url}` : `📎 Attachment: ${fileData.url}`;
        document.getElementById('lecturer-file-name').classList.add('hidden');
      }

      if (!content || !activeStudentId) return;

      await supabaseClient.from('messages').insert({
        student_id: activeStudentId,
        sender_id: currentUser.id,
        content: content
      });

      input.value = '';
    }

    /* --- STUDENT DASHBOARD --- */
    async function loadStudentMessages() {
      const { data: messages } = await supabaseClient
        .from('messages')
        .select('*')
        .eq('student_id', currentUser.id)
        .order('created_at', { ascending: true });

      const speedData = calculateAvgResponseSpeed(messages || [], currentUser.id);
      const badge = document.getElementById('student-speed-badge');
      if (badge) {
        badge.innerText = speedData.label;
        badge.className = `text-[10px] px-2 py-0.5 rounded-full font-bold ${speedData.color}`;
      }

      renderMessages(messages || [], 'student-messages-box');
    }

    async function sendStudentMessage() {
      const input = document.getElementById('student-message-input');
      let content = input.value.trim();
      const fileData = await uploadAttachment('student-file-input');

      if (fileData) {
        content = content ? `${content}\n\n📎 Attachment: ${fileData.url}` : `📎 Attachment: ${fileData.url}`;
        document.getElementById('student-file-name').classList.add('hidden');
      }

      if (!content) return;

      if (content.length >= 10) {
        startCountdownTimer();
      } else {
        alert("⚠️ Message sent, but it was under 10 characters! Your 2-minute survival timer was NOT reset.");
      }

      await supabaseClient.from('messages').insert({
        student_id: currentUser.id,
        sender_id: currentUser.id,
        content: content
      });

      input.value = '';
    }

    /* --- SHARED RENDER & DELETE LOGIC --- */
    function renderMessages(messages, containerId) {
      const box = document.getElementById(containerId);
      box.innerHTML = '';

      messages.forEach(msg => {
        if (msg.deleted_by && msg.deleted_by.includes(currentUser.id)) return;

        const isMe = msg.sender_id === currentUser.id;
        const msgDiv = document.createElement('div');
        msgDiv.className = `flex flex-col ${isMe ? 'items-end' : 'items-start'}`;

        let contentText = msg.deleted_for_everyone 
          ? '<i class="opacity-60">This message was deleted</i>' 
          : formatMessageText(msg.content);

        const bgStyle = containerId === 'lecturer-messages-box' 
          ? (isMe ? 'bg-gradient-to-r from-cyan-500 to-pink-500 text-white' : 'bg-zinc-800 text-zinc-200 border border-zinc-700')
          : (isMe ? 'bg-blue-600 text-white' : 'bg-zinc-800 text-zinc-100 border border-zinc-700');

        msgDiv.innerHTML = `
          <div class="max-w-[85%] md:max-w-md p-2.5 rounded-2xl text-xs relative group ${bgStyle}">
            <p class="whitespace-pre-wrap leading-relaxed">${contentText}</p>
            ${!msg.deleted_for_everyone ? `
              <div class="hidden group-hover:flex gap-2 mt-1.5 text-[9px] opacity-80 border-t border-white/20 pt-1">
                <button onclick="deleteMessage('${msg.id}', 'me')" class="underline hover:text-red-300">Delete for me</button>
                <button onclick="deleteMessage('${msg.id}', 'everyone')" class="underline hover:text-red-300">Delete for everyone</button>
              </div>
            ` : ''}
          </div>
        `;
        box.appendChild(msgDiv);
      });
      box.scrollTop = box.scrollHeight;
    }

    async function deleteMessage(msgId, mode) {
      if (mode === 'everyone') {
        await supabaseClient.from('messages').update({ deleted_for_everyone: true }).eq('id', msgId);
      } else if (mode === 'me') {
        const { data: msg } = await supabaseClient.from('messages').select('deleted_by').eq('id', msgId).single();
        const updatedDeletedBy = [...(msg.deleted_by || []), currentUser.id];
        await supabaseClient.from('messages').update({ deleted_by: updatedDeletedBy }).eq('id', msgId);
      }

      if (userProfile.role === 'lecturer') {
        loadLecturerMessages();
      } else {
        loadStudentMessages();
      }
    }

    function subscribeToMessages() {
      supabaseClient
        .channel('messages_realtime')
        .on('postgres_changes', { event: '*', schema: 'public', table: 'messages' }, () => {
          if (userProfile.role === 'lecturer') {
            loadTikTokStudentFeed();
            loadLecturerMessages();
          } else {
            loadStudentMessages();
          }
        })
        .subscribe();
    }
  </script>
</body>
</html>
