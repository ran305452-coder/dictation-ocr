[dictation_system.html](https://github.com/user-attachments/files/26918024/dictation_system.html)
[ocr_server.py](https://github.com/user-attachments/files/26918025/ocr_server.py)

web: python ocr_server.py

flask
flask-cors
paddleocr==2.7.3
paddlepaddle==3.0.0
numpy==1.26.4
Pillow
opencv-python-headless

<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>國語聽寫</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700;900&display=swap');
:root{
  --bg:#F7F3EE;--white:#fff;--ink:#1C1917;--muted:#9C9189;--border:#E5DDD5;
  --orange:#F97316;--od:#C2550E;--ol:#FEF0E7;
  --green:#22C55E;--gd:#15803D;--gl:#DCFCE7;
  --red:#EF4444;--rd:#B91C1C;--rl:#FEE2E2;
  --yellow:#EAB308;--yl:#FEF9C3;
  --blue:#3B82F6;--bl:#EFF6FF;--bd:#1D4ED8;
  --r:16px;--rlg:24px;
  --sh:0 2px 12px rgba(0,0,0,.08);--shlg:0 8px 32px rgba(0,0,0,.12);
}
*{box-sizing:border-box;margin:0;padding:0;-webkit-tap-highlight-color:transparent;}
html,body{height:100%;overflow:hidden;}
body{font-family:'Noto Sans TC',sans-serif;background:var(--bg);color:var(--ink);}
.page{display:none;height:100dvh;flex-direction:column;overflow:hidden;}
.page.on{display:flex;}
.scroll{overflow-y:auto;flex:1;}

/* Header */
.hd{background:var(--white);border-bottom:1.5px solid var(--border);
  padding:12px 18px;display:flex;align-items:center;gap:10px;flex-shrink:0;}
.hd-title{font-size:19px;font-weight:900;flex:1;}
.back{width:38px;height:38px;border-radius:50%;background:rgba(0,0,0,.07);
  border:none;font-size:19px;cursor:pointer;display:flex;align-items:center;
  justify-content:center;flex-shrink:0;}

/* Buttons */
.btn{font-family:'Noto Sans TC',sans-serif;font-weight:700;border:none;cursor:pointer;
  border-radius:var(--r);transition:transform .12s,box-shadow .12s;
  display:inline-flex;align-items:center;justify-content:center;gap:6px;}
.btn:active{transform:scale(.96);}
.bl{font-size:19px;padding:15px 20px;width:100%;}
.bm{font-size:16px;padding:11px 18px;}
.bs{font-size:14px;padding:8px 14px;}
.bo{background:var(--orange);color:#fff;box-shadow:0 4px 0 var(--od);}
.bo:active{box-shadow:0 1px 0 var(--od);transform:translateY(3px);}
.bg{background:var(--green);color:#fff;box-shadow:0 4px 0 var(--gd);}
.bg:active{box-shadow:0 1px 0 var(--gd);transform:translateY(3px);}
.br{background:var(--red);color:#fff;box-shadow:0 4px 0 var(--rd);}
.br:active{box-shadow:0 1px 0 var(--rd);transform:translateY(3px);}
.by{background:var(--yellow);color:#fff;box-shadow:0 4px 0 #a16207;}
.by:active{box-shadow:0 1px 0 #a16207;transform:translateY(3px);}
.bb{background:var(--blue);color:#fff;box-shadow:0 4px 0 var(--bd);}
.bb:active{box-shadow:0 1px 0 var(--bd);transform:translateY(3px);}
.bgh{background:var(--white);color:var(--ink);border:2px solid var(--border);}

/* Input */
.inp{font-family:'Noto Sans TC',sans-serif;font-size:16px;padding:11px 14px;
  border:2px solid var(--border);border-radius:var(--r);background:var(--white);
  color:var(--ink);width:100%;outline:none;transition:border-color .15s;}
.inp:focus{border-color:var(--orange);}

/* Card */
.card{background:var(--white);border-radius:var(--rlg);padding:18px;box-shadow:var(--sh);}
.ct{font-size:16px;font-weight:700;margin-bottom:12px;}

/* Chips */
.chips{display:flex;flex-wrap:wrap;gap:8px;}
.chip{padding:7px 14px;border-radius:50px;border:2px solid var(--border);
  font-size:14px;font-weight:500;cursor:pointer;background:var(--white);
  font-family:'Noto Sans TC',sans-serif;transition:all .15s;}
.chip.on{background:var(--orange);border-color:var(--orange);color:#fff;}

/* Spinner */
.spin{border-radius:50%;animation:sp .7s linear infinite;}
@keyframes sp{to{transform:rotate(360deg);}}

/* Toast */
#toast{position:fixed;bottom:24px;left:50%;transform:translateX(-50%) translateY(80px);
  background:#1C1917;color:#fff;padding:11px 22px;border-radius:50px;
  font-size:14px;font-weight:500;font-family:'Noto Sans TC',sans-serif;
  transition:transform .25s;z-index:9999;pointer-events:none;white-space:nowrap;}
#toast.show{transform:translateX(-50%) translateY(0);}

/* ══ HOME ══ */
#pg-home{align-items:center;justify-content:center;gap:24px;padding:28px 20px;
  background:linear-gradient(160deg,#FFF9F4 0%,#FEE9D7 100%);}
.home-ico{font-size:72px;animation:bob 2.4s ease-in-out infinite;}
@keyframes bob{0%,100%{transform:translateY(0);}50%{transform:translateY(-8px);}}
.home-title{font-size:34px;font-weight:900;color:var(--orange);letter-spacing:-1px;}
.home-sub{font-size:16px;color:var(--muted);}
.home-card{background:var(--white);border-radius:var(--rlg);padding:20px;
  width:100%;max-width:400px;box-shadow:var(--shlg);display:flex;flex-direction:column;gap:12px;}
.home-btns{display:flex;flex-direction:column;gap:10px;width:100%;max-width:400px;}

/* Student list on home */
.stu-list{display:flex;flex-direction:column;gap:6px;max-height:160px;overflow-y:auto;}
.stu-row{display:flex;align-items:center;gap:8px;padding:8px 12px;
  border-radius:var(--r);border:2px solid var(--border);cursor:pointer;
  background:var(--white);transition:border-color .15s;}
.stu-row:hover{border-color:var(--orange);}
.stu-avatar{width:36px;height:36px;border-radius:50%;background:var(--ol);
  display:flex;align-items:center;justify-content:center;
  font-size:16px;font-weight:900;color:var(--od);flex-shrink:0;}
.stu-name{font-size:16px;font-weight:700;flex:1;}
.stu-meta{font-size:12px;color:var(--muted);}

/* ══ SETUP ══ */
#pg-setup{overflow:hidden;}
.setup-body{padding:18px;display:flex;flex-direction:column;gap:14px;}

/* Lesson grid */
.lesson-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:8px;}
.lesson-btn{padding:10px 4px;border-radius:12px;border:2px solid var(--border);
  font-size:14px;font-weight:700;cursor:pointer;background:var(--white);
  font-family:'Noto Sans TC',sans-serif;text-align:center;transition:all .15s;}
.lesson-btn.on{background:var(--orange);border-color:var(--orange);color:#fff;}
.lesson-btn.has-words{border-color:var(--green);color:var(--gd);}
.lesson-btn.on.has-words{background:var(--orange);border-color:var(--orange);color:#fff;}

/* Word editor */
.word-editor{background:var(--bg);border-radius:var(--r);padding:14px;margin-top:10px;}
.we-title{font-size:14px;font-weight:700;color:var(--muted);margin-bottom:8px;}
.word-tags{display:flex;flex-wrap:wrap;gap:6px;min-height:32px;}
.wtag{background:var(--ol);color:var(--od);border-radius:6px;
  padding:4px 8px;font-size:15px;font-weight:700;}
.wtag .x{opacity:.5;margin-left:3px;font-size:12px;cursor:pointer;}
.word-ta{min-height:80px;resize:none;font-size:16px;margin-top:8px;}

/* Upload zone */
.upload-z{border:3px dashed var(--border);border-radius:var(--r);
  padding:18px;text-align:center;cursor:pointer;}
.upload-z:hover{border-color:var(--orange);}
.img-pre{width:100%;max-height:140px;object-fit:contain;border-radius:var(--r);
  margin-top:8px;display:none;}
.ocr-spin{display:none;align-items:center;gap:8px;padding:10px;
  background:var(--yl);border-radius:var(--r);font-size:14px;color:#854d0e;margin-top:6px;}

/* Server card */
.sv-card{border:2px solid var(--orange);}
.sv-status{margin-top:8px;font-size:13px;padding:7px 11px;
  border-radius:var(--r);background:var(--bg);display:none;}

/* ══ STUDENT MANAGER ══ */
#pg-students{overflow:hidden;}
.stu-manager-body{padding:18px;display:flex;flex-direction:column;gap:12px;}
.stu-big-card{background:var(--white);border-radius:var(--rlg);
  padding:18px;box-shadow:var(--sh);}
.stu-full-row{display:flex;align-items:center;gap:10px;padding:12px;
  border-radius:var(--r);border:1.5px solid var(--border);cursor:pointer;}
.stu-full-row:hover{border-color:var(--orange);background:var(--ol);}
.stu-full-avatar{width:44px;height:44px;border-radius:50%;
  display:flex;align-items:center;justify-content:center;
  font-size:20px;font-weight:900;flex-shrink:0;}
.stu-info{flex:1;}
.stu-info-name{font-size:17px;font-weight:900;}
.stu-info-meta{font-size:12px;color:var(--muted);margin-top:1px;}
.stu-info-score{font-size:18px;font-weight:900;}

/* ══ STUDENT PROFILE ══ */
#pg-profile{overflow:hidden;}
.profile-body{padding:18px;display:flex;flex-direction:column;gap:14px;}
.profile-hero{background:linear-gradient(135deg,var(--orange) 0%,var(--od) 100%);
  border-radius:var(--rlg);padding:22px;color:#fff;text-align:center;}
.profile-avatar{width:70px;height:70px;border-radius:50%;background:rgba(255,255,255,.25);
  display:flex;align-items:center;justify-content:center;
  font-size:32px;font-weight:900;margin:0 auto 10px;}
.profile-name{font-size:24px;font-weight:900;}
.profile-stats{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-top:14px;}
.profile-stat{background:rgba(255,255,255,.2);border-radius:10px;padding:10px;text-align:center;}
.profile-stat-num{font-size:24px;font-weight:900;}
.profile-stat-label{font-size:11px;opacity:.85;margin-top:1px;}
.weak-words{display:flex;flex-wrap:wrap;gap:6px;}
.weak-chip{padding:5px 10px;border-radius:6px;font-size:15px;font-weight:700;cursor:pointer;}
.wc-ng{background:var(--rl);color:var(--rd);}
.wc-ok{background:var(--gl);color:var(--gd);}
.history-item{background:var(--white);border-radius:var(--r);padding:12px 14px;
  border:1.5px solid var(--border);}
.hi-top{display:flex;justify-content:space-between;align-items:center;margin-bottom:8px;}
.hi-date{font-size:12px;color:var(--muted);}
.hi-score{font-size:16px;font-weight:900;}
.hi-chips{display:flex;flex-wrap:wrap;gap:5px;}
.hi-chip{padding:3px 8px;border-radius:5px;font-size:14px;font-weight:700;}

/* ══ DICTATION ══ */
#pg-dict{background:#F0EBE3;}
.dict-ctrl{background:var(--white);border-bottom:1.5px solid var(--border);
  padding:10px 12px;display:flex;flex-direction:column;gap:8px;flex-shrink:0;}
.dict-top{display:flex;align-items:center;gap:8px;}
.d-qnum{font-size:20px;font-weight:900;background:var(--orange);color:#fff;
  padding:3px 12px;border-radius:50px;flex-shrink:0;}
.d-prog{flex:1;font-size:15px;color:var(--muted);font-weight:500;}
.d-name{font-size:14px;font-weight:700;color:var(--orange);}
.dict-btns{display:grid;grid-template-columns:repeat(4,1fr);gap:7px;}
.dict-btns .btn{font-size:15px;padding:12px 4px;border-radius:11px;}
.dict-write{flex:1;position:relative;margin:10px 10px 0;
  border-radius:var(--rlg);background:var(--white);
  box-shadow:var(--shlg);overflow:hidden;min-height:0;}
.write-cvs{position:absolute;inset:0;width:100%;height:100%;
  cursor:crosshair;touch-action:none;}
.write-hint{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);
  font-size:18px;color:rgba(0,0,0,.12);font-weight:700;pointer-events:none;
  user-select:none;text-align:center;line-height:1.8;}
.dict-submit{padding:8px 10px 12px;flex-shrink:0;}

/* ══ SCORING ══ */
#pg-score{align-items:center;justify-content:center;padding:28px 20px;
  background:linear-gradient(160deg,#F7F3EE 0%,#FEE9D7 100%);}
.score-card{background:var(--white);border-radius:var(--rlg);padding:28px;
  width:100%;max-width:440px;box-shadow:var(--shlg);
  display:flex;flex-direction:column;align-items:center;gap:18px;}

/* ══ RESULT ══ */
#pg-result{align-items:center;justify-content:center;padding:20px;
  background:linear-gradient(160deg,#F7F3EE 0%,#FEE9D7 100%);}
.result-card{background:var(--white);border-radius:var(--rlg);padding:22px;
  width:100%;max-width:460px;box-shadow:var(--shlg);
  display:flex;flex-direction:column;gap:14px;}
.verdict{display:flex;align-items:center;gap:12px;padding:14px;border-radius:var(--r);}
.vi{font-size:44px;line-height:1;flex-shrink:0;}
.vword{font-size:34px;font-weight:900;}
.vmsg{font-size:15px;margin-top:2px;}
.bar-wrap{background:var(--bg);border-radius:8px;height:12px;overflow:hidden;}
.bar-fill{height:100%;border-radius:8px;transition:width .6s ease;}
.bk-row{display:flex;justify-content:space-between;align-items:center;font-size:15px;}
.bk-l{color:var(--muted);}
.bk-v{font-weight:700;}
.res-acts{display:flex;flex-direction:column;gap:8px;}

/* ══ PRACTICE ══ */
#pg-practice{background:#F0EBE3;}
.prac-hd{background:var(--green);color:#fff;padding:12px 18px;flex-shrink:0;
  display:flex;align-items:center;gap:8px;}
.prac-hd-title{font-size:19px;font-weight:900;flex:1;}
.prac-chars-row{flex-shrink:0;display:flex;align-items:center;
  justify-content:center;padding:10px 14px;gap:8px;}
.pchar{width:64px;height:64px;border-radius:12px;background:var(--white);
  box-shadow:var(--sh);display:flex;align-items:center;justify-content:center;
  font-size:38px;font-weight:900;border:3px solid var(--border);}
.pchar.cur{border-color:var(--green);background:var(--gl);}
.pchar.done{border-color:var(--green);opacity:.5;}
.prac-write{flex:1;position:relative;margin:0 10px;
  border-radius:var(--rlg);background:var(--white);
  box-shadow:var(--shlg);overflow:hidden;min-height:0;}
.prac-ghost{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);
  font-size:clamp(110px,32vw,190px);font-weight:900;color:rgba(249,115,22,.12);
  pointer-events:none;user-select:none;line-height:1;
  font-family:'Noto Sans TC',sans-serif;
  text-shadow:0 0 0 rgba(249,115,22,.08);}
.prac-cvs{position:absolute;inset:0;width:100%;height:100%;
  cursor:crosshair;touch-action:none;}
.prac-btns{padding:10px 12px 14px;flex-shrink:0;display:flex;gap:8px;}
.prac-btns .btn{flex:1;font-size:16px;padding:13px;}

/* ══ FINAL ══ */
#pg-final{overflow:hidden;}
.final-hd{background:var(--orange);color:#fff;padding:18px 20px;
  flex-shrink:0;text-align:center;}
.final-hd h2{font-size:26px;font-weight:900;}
.final-hd p{font-size:14px;opacity:.85;margin-top:2px;}
.final-body{padding:18px;display:flex;flex-direction:column;gap:12px;}
.final-score-card{background:var(--white);border-radius:var(--rlg);
  padding:22px;box-shadow:var(--sh);text-align:center;}
.final-big{font-size:80px;font-weight:900;line-height:1;}
.final-grade{display:inline-block;padding:5px 20px;border-radius:50px;
  font-size:18px;font-weight:900;margin-top:6px;}
.wri-item{background:var(--white);border-radius:var(--rlg);
  padding:12px 18px;box-shadow:var(--sh);
  display:flex;align-items:center;gap:12px;}
.wri-ic{font-size:24px;flex-shrink:0;}
.wri-word{font-size:24px;font-weight:900;flex:1;}
.wri-sc{font-size:18px;font-weight:900;}
.final-acts{display:flex;flex-direction:column;gap:10px;padding-bottom:20px;}
</style>
</head>
<body>

<!-- ══ HOME ══ -->
<div id="pg-home" class="page on">
  <div class="home-ico">📝</div>
  <div style="text-align:center">
    <div class="home-title">國語聽寫</div>
    <div class="home-sub">1～2年級專用</div>
  </div>

  <!-- 學生選擇 / 新增 -->
  <div class="home-card">
    <div style="font-size:16px;font-weight:700;">👤 選擇學生或輸入姓名</div>
    <div class="stu-list" id="home-stu-list"></div>
    <input id="inp-name" class="inp" placeholder="輸入新姓名..." maxlength="10"
      style="font-size:20px;font-weight:700;">
  </div>

  <div class="home-btns">
    <button class="btn bo bl" onclick="goTest()">✏️ 開始聽寫</button>
    <button class="btn bb bl" onclick="goStudents()">👥 學生管理</button>
    <button class="btn bgh bl" onclick="goSetup()">⚙️ 老師設定</button>
  </div>
</div>

<!-- ══ SETUP ══ -->
<div id="pg-setup" class="page">
  <div class="hd">
    <button class="back" onclick="goPage('pg-home')">←</button>
    <div class="hd-title">老師設定</div>
  </div>
  <div class="scroll">
  <div class="setup-body">

    <!-- 年級 & 版本 -->
    <div class="card">
      <div class="ct">📚 年級與版本</div>
      <div class="chips" id="chips-grade">
        <div class="chip on" data-g="grade" data-v="1">一年級</div>
        <div class="chip" data-g="grade" data-v="2">二年級</div>
      </div>
      <div class="chips" style="margin-top:8px;" id="chips-pub">
        <div class="chip on" data-g="pub" data-v="翰林">翰林</div>
        <div class="chip" data-g="pub" data-v="南一">南一</div>
        <div class="chip" data-g="pub" data-v="康軒">康軒</div>
      </div>
    </div>

    <!-- 課次管理 -->
    <div class="card">
      <div class="ct">📖 選擇課次（點課次編輯詞語）</div>
      <div class="lesson-grid" id="lesson-grid"></div>
      <div class="word-editor" id="word-editor" style="display:none;">
        <div class="we-title" id="we-title">第1課 詞語</div>
        <div class="word-tags" id="we-tags"></div>
        <textarea id="we-ta" class="inp word-ta"
          placeholder="輸入詞語，一行一個&#10;天空&#10;花朵"></textarea>
        <div style="display:flex;gap:8px;margin-top:8px;">
          <button class="btn bo bs" onclick="saveLesson()">✓ 儲存本課</button>
          <button class="btn bgh bs" onclick="closeEditor()">取消</button>
        </div>
      </div>
    </div>

    <!-- 圖片辨識 -->
    <div class="card">
      <div class="ct">📷 拍照上傳辨識詞語</div>
      <div class="upload-z" onclick="document.getElementById('img-inp').click()">
        <div style="font-size:36px;margin-bottom:4px;">📸</div>
        <div style="font-size:14px;color:var(--muted);">點這裡拍照或選圖片</div>
      </div>
      <input type="file" id="img-inp" accept="image/*" capture="environment" style="display:none">
      <img id="img-pre" class="img-pre">
      <div id="ocr-spin" class="ocr-spin">
        <div class="spin" style="width:18px;height:18px;border:3px solid rgba(133,77,14,.25);border-top-color:#854d0e;flex-shrink:0;"></div>
        <span>OCR 辨識中...</span>
      </div>
      <div style="font-size:13px;color:var(--muted);margin-top:8px;">
        辨識到的詞語會加入目前選擇的課次
      </div>
    </div>

    <!-- 播放設定 -->
    <div class="card">
      <div class="ct">🔊 播放設定</div>
      <label style="display:flex;align-items:center;justify-content:space-between;font-size:15px;">
        說話速度
        <select id="sel-rate" class="inp" style="width:auto;font-size:15px;padding:7px 11px;">
          <option value="0.6">慢速</option>
          <option value="0.8" selected>普通</option>
          <option value="1.0">稍快</option>
        </select>
      </label>
    </div>

    <!-- OCR 伺服器 -->
    <div class="card sv-card">
      <div class="ct">🖥️ PaddleOCR 辨識伺服器</div>
      <p style="font-size:12px;color:var(--muted);line-height:1.7;margin-bottom:8px;">
        啟動方式：<code style="background:var(--bg);padding:1px 5px;border-radius:4px;">python ocr_server.py</code>
      </p>
      <input id="inp-server" class="inp" placeholder="http://localhost:5000"
        style="font-size:14px;margin-bottom:8px;">
      <div style="display:flex;gap:7px;">
        <button class="btn by bs" onclick="saveServer()">儲存</button>
        <button class="btn bgh bs" onclick="testServer()">測試連線</button>
      </div>
      <div id="sv-status" class="sv-status"></div>
    </div>

    <button class="btn bo bl" style="margin-bottom:20px;" onclick="saveSetup()">💾 儲存設定</button>
  </div>
  </div>
</div>

<!-- ══ STUDENT MANAGER ══ -->
<div id="pg-students" class="page">
  <div class="hd">
    <button class="back" onclick="goPage('pg-home')">←</button>
    <div class="hd-title">學生管理</div>
    <button class="btn bo bs" onclick="addNewStudent()">➕ 新增</button>
  </div>
  <div class="scroll">
  <div class="stu-manager-body" id="stu-manager-body">
  </div>
  </div>
</div>

<!-- ══ STUDENT PROFILE ══ -->
<div id="pg-profile" class="page">
  <div class="hd">
    <button class="back" onclick="goPage('pg-students')">←</button>
    <div class="hd-title" id="profile-hd-title">學生檔案</div>
    <button class="btn br bs" id="btn-del-stu" onclick="deleteStudent()">刪除</button>
  </div>
  <div class="scroll">
  <div class="profile-body" id="profile-body">
  </div>
  </div>
</div>

<!-- ══ DICTATION ══ -->
<div id="pg-dict" class="page">
  <div class="dict-ctrl">
    <div class="dict-top">
      <div class="d-qnum" id="d-qnum">第1題</div>
      <div class="d-prog" id="d-prog">1/5</div>
      <div class="d-name" id="d-name">小明</div>
    </div>
    <div class="dict-btns" style="grid-template-columns:repeat(3,1fr);">
      <button class="btn by" id="btn-play" onclick="playWord()">🔊 播放</button>
      <button class="btn bgh" onclick="clearWrite()">🗑️ 清除</button>
      <button class="btn bgh" onclick="skipWord()" style="font-size:13px;">🙅 不會</button>
    </div>
  </div>
  <div class="dict-write" id="dict-write">
    <canvas id="write-cvs" class="write-cvs"></canvas>
    <div class="write-hint" id="write-hint">🎧 先聽題<br>再在這裡寫字</div>
  </div>
  <div class="dict-submit">
    <button class="btn bg bl" onclick="submitAnswer()">✓ 確認送出</button>
  </div>
</div>

<!-- ══ SCORING ══ -->
<div id="pg-score" class="page">
  <div class="score-card">
    <div class="spin" style="width:60px;height:60px;border:6px solid var(--ol);border-top-color:var(--orange);"></div>
    <div style="font-size:20px;font-weight:900;">AI 判斷中...</div>
    <div style="font-size:15px;color:var(--muted);">請稍候</div>
  </div>
</div>

<!-- ══ RESULT ══ -->
<div id="pg-result" class="page">
  <div class="result-card" id="result-card"></div>
</div>

<!-- ══ PRACTICE ══ -->
<div id="pg-practice" class="page">
  <div class="prac-hd">
    <span class="prac-hd-title">✍️ 補強練習</span>
    <span id="prac-lbl" style="font-size:15px;opacity:.85;"></span>
  </div>
  <!-- 正確字展示區 -->
  <div style="background:white;border-bottom:1.5px solid var(--border);
    padding:10px 16px;flex-shrink:0;display:flex;align-items:center;gap:12px;">
    <div style="font-size:13px;color:var(--muted);flex-shrink:0;">正確字：</div>
    <div id="prac-answer-display" style="font-size:36px;font-weight:900;
      color:var(--orange);letter-spacing:8px;"></div>
    <div style="flex:1;font-size:12px;color:var(--muted);line-height:1.5;">
      照著橘色底字描寫<br>可清除重寫
    </div>
  </div>
  <div class="prac-chars-row" id="prac-chars-row"></div>
  <div class="prac-write" id="prac-write">
    <div class="prac-ghost" id="prac-ghost"></div>
    <canvas id="prac-cvs" class="prac-cvs"></canvas>
  </div>
  <div class="prac-btns">
    <button class="btn br" onclick="clearPrac()">🗑️ 清除重寫</button>
    <button class="btn bg" onclick="nextPrac()">寫好了 →</button>
  </div>
</div>

<!-- ══ FINAL ══ -->
<div id="pg-final" class="page">
  <div class="final-hd">
    <h2>🎉 完成！</h2>
    <p id="final-name"></p>
  </div>
  <div class="scroll">
  <div class="final-body">
    <div class="final-score-card">
      <div class="final-big" id="final-score"></div>
      <div style="font-size:16px;color:var(--muted);">分</div>
      <div class="final-grade" id="final-grade"></div>
    </div>
    <div id="final-list"></div>
    <div class="final-acts">
      <button class="btn br bl" id="btn-retry" onclick="retryWrong()">🔁 重考錯誤詞語</button>
      <button class="btn bgh bl" onclick="()=>{goStudents();}">👥 查看學生紀錄</button>
      <button class="btn bgh bl" onclick="goPage('pg-home')">🏠 回首頁</button>
    </div>
  </div>
  </div>
</div>

<div id="toast"></div>

<script>
// ══════════════════════════════════════════
//  DATA LAYER
// ══════════════════════════════════════════

// ─ Lessons DB ─
// Structure: { '翰林': { '1': { '1': ['詞語',...], '2': [...], ... }, '2': {...} } }
const DEFAULT_LESSONS = {
  '翰林':{
    '1':{
      '1':['大小','上下','左右','前後'],
      '2':['日月','山水','天地','人口'],
      '3':['父母','兄弟','姐妹','朋友'],
      '4':['春天','夏天','秋天','冬天'],
      '5':['花草','樹木','飛鳥','魚蟲'],
      '6':['早起','讀書','上學','回家'],
      '7':['快樂','美好','開心','高興'],
      '8':['吃飯','喝水','睡覺','洗澡'],
      '9':['太陽','月亮','星星','雲朵'],
      '10':['紅色','黃色','藍色','綠色'],
      '11':['小狗','貓咪','兔子','小鳥'],
      '12':['一心','二手','三口','四方'],
    },
    '2':{
      '1':['美麗','快樂','可愛','聰明'],
      '2':['圖書館','操場','教室','走廊'],
      '3':['運動','唱歌','跳舞','畫畫'],
      '4':['誠實','勇敢','善良','努力'],
      '5':['感謝','道歉','幫助','分享'],
      '6':['春風','細雨','彩虹','白雪'],
      '7':['高山','大海','草原','森林'],
      '8':['老師','同學','父母','家人'],
      '9':['認真','用心','努力','加油'],
      '10':['蝴蝶','螞蟻','蜜蜂','蜻蜓'],
      '11':['早晨','中午','傍晚','夜晚'],
      '12':['世界','國家','城市','鄉村'],
    }
  },
  '南一':{
    '1':{'1':['太陽','月亮','星星','天空'],'2':['山丘','河流','大海','沙灘'],
         '3':['小狗','貓咪','兔子','松鼠'],'4':['蘋果','香蕉','西瓜','葡萄'],
         '5':['開心','難過','生氣','害怕'],'6':['跑步','跳繩','游泳','騎車'],
         '7':['媽媽','爸爸','爺爺','奶奶'],'8':['教室','圖書館','餐廳','廁所'],
         '9':['鉛筆','橡皮擦','尺','書包'],'10':['一二三','四五六','七八九','十百千'],
         '11':['紅橙黃','綠藍紫','黑白灰','彩色'],'12':['東南西','北前後','左右上','下裡外']},
    '2':{'1':['勇敢','誠實','禮貌','謙虛'],'2':['春天','夏天','秋天','冬天'],
         '3':['颱風','地震','洪水','乾旱'],'4':['民主','自由','平等','博愛'],
         '5':['環保','回收','節能','愛地球'],'6':['傳統','文化','習俗','節日'],
         '7':['交通','公車','捷運','高鐵'],'8':['市場','超市','百貨','便利店'],
         '9':['數學','自然','社會','美術'],'10':['運動會','園遊會','畢業典禮','升旗典禮'],
         '11':['電腦','手機','平板','電視'],'12':['夢想','目標','計畫','行動']}
  },
  '康軒':{
    '1':{'1':['一二三四','五六七八','九十百千','萬億兆'],'2':['東西南北','春夏秋冬','金木水火','天地人和'],
         '3':['貓狗牛羊','雞鴨鵝豬','馬驢駱駝','獅虎豹熊'],'4':['梅蘭竹菊','松柏楊柳','玫瑰荷花','牡丹桂花'],
         '5':['日出日落','月圓月缺','星光閃閃','雲捲雲舒'],'6':['笑口常開','喜上眉梢','愁眉苦臉','怒髮衝冠'],
         '7':['山明水秀','風和日麗','鳥語花香','天高氣爽'],'8':['同心協力','互相幫助','分工合作','齊心努力'],
         '9':['求知若渴','勤學苦練','博覽群書','學以致用'],'10':['尊師重道','敬老愛幼','孝順父母','友愛兄弟'],
         '11':['誠信守約','言行一致','知錯能改','善莫大焉'],'12':['愛護環境','珍惜資源','節約能源','永續發展']},
    '2':{'1':['安全','健康','快樂','幸福'],'2':['創意','發明','設計','製作'],
         '3':['歌唱','跳舞','繪畫','演戲'],'4':['閱讀','寫作','演講','討論'],
         '5':['旅行','探險','觀察','記錄'],'6':['種植','飼養','照顧','呵護'],
         '7':['合作','競爭','尊重','包容'],'8':['時間','空間','數量','品質'],
         '9':['過去','現在','未來','歷史'],'10':['原住民','客家人','閩南人','新住民'],
         '11':['地球','月球','太陽','宇宙'],'12':['友情','親情','愛情','師生情']}
  }
};

function getLessonsDB() {
  try { return JSON.parse(localStorage.getItem('lessons_db') || 'null') || DEFAULT_LESSONS; }
  catch(e) { return DEFAULT_LESSONS; }
}
function saveLessonsDB(db) {
  localStorage.setItem('lessons_db', JSON.stringify(db));
}

// ─ Students DB ─
function getStudents() {
  try { return JSON.parse(localStorage.getItem('students') || '[]'); }
  catch(e) { return []; }
}
function saveStudents(arr) {
  localStorage.setItem('students', JSON.stringify(arr));
}
function getStudent(name) {
  return getStudents().find(s => s.name === name) || null;
}
function upsertStudent(name) {
  const arr = getStudents();
  if (!arr.find(s => s.name === name)) {
    arr.push({ name, created: new Date().toLocaleDateString('zh-TW'), records: [] });
    saveStudents(arr);
  }
}
function addRecord(name, rec) {
  const arr = getStudents();
  const s = arr.find(x => x.name === name);
  if (s) { s.records = s.records || []; s.records.push(rec); }
  saveStudents(arr);
}

// ══════════════════════════════════════════
//  APP STATE
// ══════════════════════════════════════════
const S = {
  name: '', grade: '1', pub: '翰林',
  selLesson: null, editLesson: null,
  queue: [], idx: 0, results: [],
  rate: 0.8, server: 'http://localhost:5000',
  pracQueue: [], pracIdx: 0, lastResult: null,
  viewStudent: null,
};

// ══════════════════════════════════════════
//  NAV
// ══════════════════════════════════════════
function goPage(id) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('on'));
  document.getElementById(id).classList.add('on');
}

function goSetup() {
  document.getElementById('inp-server').value = S.server;
  renderLessonGrid();
  goPage('pg-setup');
}

function goStudents() {
  renderStudentManager();
  goPage('pg-students');
}

// ══════════════════════════════════════════
//  TOAST
// ══════════════════════════════════════════
let _tt;
function toast(m, d=2400) {
  const e = document.getElementById('toast');
  e.textContent = m; e.classList.add('show');
  clearTimeout(_tt); _tt = setTimeout(() => e.classList.remove('show'), d);
}

// ══════════════════════════════════════════
//  HOME — student quick select
// ══════════════════════════════════════════
function renderHomeStudents() {
  const el = document.getElementById('home-stu-list');
  const arr = getStudents();
  if (!arr.length) { el.innerHTML = '<div style="font-size:13px;color:var(--muted);padding:4px 0;">還沒有學生，輸入姓名後開始</div>'; return; }
  el.innerHTML = arr.slice(-5).reverse().map(s => {
    const recs = s.records || [];
    const avg = recs.length ? Math.round(recs.reduce((a,r)=>a+r.score,0)/recs.length) : '-';
    const initials = s.name.slice(0,1);
    return `<div class="stu-row" onclick="selectStudent('${s.name}')">
      <div class="stu-avatar">${initials}</div>
      <div class="stu-name">${s.name}</div>
      <div class="stu-meta">共${recs.length}次 平均${avg}分</div>
    </div>`;
  }).join('');
}

function selectStudent(name) {
  document.getElementById('inp-name').value = name;
  toast(`已選擇 ${name}`);
}

// ══════════════════════════════════════════
//  SETUP — Chips
// ══════════════════════════════════════════
document.querySelectorAll('.chip').forEach(c => {
  c.addEventListener('click', () => {
    const g = c.dataset.g, v = c.dataset.v;
    document.querySelectorAll(`.chip[data-g="${g}"]`).forEach(x => x.classList.remove('on'));
    c.classList.add('on');
    if (g === 'grade') { S.grade = v; renderLessonGrid(); }
    if (g === 'pub')   { S.pub = v;   renderLessonGrid(); }
  });
});

// ── Lesson Grid ──
function renderLessonGrid() {
  const db = getLessonsDB();
  const lessons = db[S.pub]?.[S.grade] || {};
  const grid = document.getElementById('lesson-grid');
  grid.innerHTML = '';
  for (let i = 1; i <= 12; i++) {
    const key = String(i);
    const hasWords = (lessons[key]||[]).length > 0;
    const isOn = S.editLesson === key;
    const btn = document.createElement('button');
    btn.className = `lesson-btn ${isOn?'on':''} ${hasWords&&!isOn?'has-words':''}`;
    btn.textContent = `第${i}課`;
    btn.onclick = () => openEditor(key);
    grid.appendChild(btn);
  }
}

function openEditor(lessonKey) {
  S.editLesson = lessonKey;
  const db = getLessonsDB();
  const words = db[S.pub]?.[S.grade]?.[lessonKey] || [];
  document.getElementById('we-title').textContent = `第${lessonKey}課 詞語`;
  document.getElementById('we-ta').value = '';
  renderEditorTags(words);
  document.getElementById('word-editor').style.display = 'block';
  renderLessonGrid();
}

function closeEditor() {
  S.editLesson = null;
  document.getElementById('word-editor').style.display = 'none';
  renderLessonGrid();
}

function renderEditorTags(words) {
  const el = document.getElementById('we-tags');
  if (!words.length) { el.innerHTML = '<span style="color:var(--muted);font-size:14px;">尚無詞語</span>'; return; }
  el.innerHTML = words.map((w,i) =>
    `<div class="wtag">${w}<span class="x" onclick="removeEditorWord(${i})">✕</span></div>`).join('');
}

function removeEditorWord(i) {
  const db = getLessonsDB();
  const words = db[S.pub][S.grade][S.editLesson] || [];
  words.splice(i, 1);
  db[S.pub][S.grade][S.editLesson] = words;
  saveLessonsDB(db);
  renderEditorTags(words);
  renderLessonGrid();
}

function saveLesson() {
  const raw = document.getElementById('we-ta').value;
  const newWords = raw.split(/[\n\r]+/).map(w => w.trim()).filter(w => w.length >= 1);
  const db = getLessonsDB();
  if (!db[S.pub]) db[S.pub] = {};
  if (!db[S.pub][S.grade]) db[S.pub][S.grade] = {};
  const existing = db[S.pub][S.grade][S.editLesson] || [];
  newWords.forEach(w => { if (!existing.includes(w)) existing.push(w); });
  db[S.pub][S.grade][S.editLesson] = existing;
  saveLessonsDB(db);
  document.getElementById('we-ta').value = '';
  renderEditorTags(existing);
  renderLessonGrid();
  toast(`第${S.editLesson}課已儲存 ${existing.length} 個詞語`);
}

// ── Image OCR → into current lesson ──
document.getElementById('img-inp').addEventListener('change', async function(e) {
  const file = e.target.files[0]; if (!file) return;
  if (!S.editLesson) { toast('請先點選一個課次再上傳圖片'); return; }
  const reader = new FileReader();
  reader.onload = async ev => {
    document.getElementById('img-pre').src = ev.target.result;
    document.getElementById('img-pre').style.display = 'block';
    document.getElementById('ocr-spin').style.display = 'flex';
    try {
      const b64 = ev.target.result.split(',')[1];
      const resp = await fetch(`${S.server}/ocr`, {
        method: 'POST', headers: {'Content-Type':'application/json'},
        body: JSON.stringify({ image: b64, mode: 'wordlist' }),
        signal: AbortSignal.timeout(20000)
      });
      const data = await resp.json();
      const raw = (data.raw || '').split(/[\n\r\s、，,]+/).map(w => w.trim())
        .filter(w => /[\u4e00-\u9fff]/.test(w) && w.length >= 2 && w.length <= 6);
      const db = getLessonsDB();
      if (!db[S.pub]) db[S.pub] = {};
      if (!db[S.pub][S.grade]) db[S.pub][S.grade] = {};
      const existing = db[S.pub][S.grade][S.editLesson] || [];
      let added = 0;
      raw.forEach(w => { if (!existing.includes(w)) { existing.push(w); added++; } });
      db[S.pub][S.grade][S.editLesson] = existing;
      saveLessonsDB(db);
      renderEditorTags(existing);
      renderLessonGrid();
      toast(`辨識加入 ${added} 個詞語到第${S.editLesson}課`);
    } catch(err) { toast('OCR 伺服器未回應，請手動輸入'); }
    document.getElementById('ocr-spin').style.display = 'none';
  };
  reader.readAsDataURL(file);
});

function saveServer() {
  const v = (document.getElementById('inp-server').value || '').trim().replace(/\/$/,'');
  S.server = v || 'http://localhost:5000';
  localStorage.setItem('ocr_server', S.server);
  toast('✓ 已儲存');
}

async function testServer() {
  const url = (document.getElementById('inp-server').value || '').trim().replace(/\/$/,'') || S.server;
  const el = document.getElementById('sv-status');
  el.style.display = 'block'; el.style.color = '#854d0e'; el.textContent = '⏳ 測試中...';
  try {
    const r = await fetch(`${url}/health`, { signal: AbortSignal.timeout(5000) });
    const d = await r.json();
    el.style.color = 'var(--gd)'; el.textContent = `✓ 連線成功！引擎：${d.engine}`;
    S.server = url; localStorage.setItem('ocr_server', url);
  } catch(e) {
    el.style.color = 'var(--rd)'; el.textContent = '✗ 連線失敗，請確認 ocr_server.py 已啟動';
  }
}

function saveSetup() {
  S.rate = parseFloat(document.getElementById('sel-rate').value);
  toast('✓ 設定已儲存'); goPage('pg-home');
}

// ══════════════════════════════════════════
//  STUDENT MANAGER
// ══════════════════════════════════════════
const COLORS = ['var(--ol)','var(--gl)','var(--bl)','var(--yl)','var(--rl)'];
const TEXT_COLORS = ['var(--od)','var(--gd)','var(--bd)','#a16207','var(--rd)'];

function renderStudentManager() {
  const arr = getStudents();
  const body = document.getElementById('stu-manager-body');
  if (!arr.length) {
    body.innerHTML = '<div style="text-align:center;color:var(--muted);font-size:16px;padding:40px;">還沒有學生紀錄<br><br>在首頁輸入姓名開始聽寫後會自動建檔</div>';
    return;
  }
  body.innerHTML = arr.map((s, idx) => {
    const recs = s.records || [];
    const avg = recs.length ? Math.round(recs.reduce((a,r)=>a+r.score,0)/recs.length) : 0;
    const ci = idx % COLORS.length;
    // Wrong words
    const wm = {};
    recs.forEach(r => r.results?.forEach(w => { if(!w.pass) wm[w.word]=(wm[w.word]||0)+1; }));
    const topWrong = Object.entries(wm).sort((a,b)=>b[1]-a[1]).slice(0,3).map(([w])=>w).join('、');
    const scoreColor = avg>=80?'var(--gd)':avg>=60?'#a16207':'var(--rd)';
    return `<div class="stu-full-row" onclick="viewStudent('${s.name}')">
      <div class="stu-full-avatar" style="background:${COLORS[ci]};color:${TEXT_COLORS[ci]}">
        ${s.name.slice(0,1)}
      </div>
      <div class="stu-info">
        <div class="stu-info-name">${s.name}</div>
        <div class="stu-info-meta">測驗${recs.length}次${topWrong?'・常錯：'+topWrong:''}</div>
      </div>
      <div class="stu-info-score" style="color:${scoreColor}">${avg?avg+'分':'-'}</div>
    </div>`;
  }).join('');
}

function addNewStudent() {
  const name = prompt('請輸入學生姓名：');
  if (!name || !name.trim()) return;
  upsertStudent(name.trim());
  renderStudentManager();
  renderHomeStudents();
  toast(`✓ 已新增學生：${name.trim()}`);
}

function viewStudent(name) {
  S.viewStudent = name;
  const s = getStudent(name);
  if (!s) return;
  document.getElementById('profile-hd-title').textContent = `${name} 的檔案`;
  renderProfile(s);
  goPage('pg-profile');
}

function renderProfile(s) {
  const recs = s.records || [];
  const body = document.getElementById('profile-body');
  const avg = recs.length ? Math.round(recs.reduce((a,r)=>a+r.score,0)/recs.length) : 0;
  const best = recs.length ? Math.max(...recs.map(r=>r.score)) : 0;

  // Wrong word map
  const wm = {};
  recs.forEach(r => r.results?.forEach(w => { if(!w.pass) wm[w.word]=(wm[w.word]||0)+1; }));
  const topWrong = Object.entries(wm).sort((a,b)=>b[1]-a[1]).slice(0,8);

  // Correct words
  const cm = {};
  recs.forEach(r => r.results?.forEach(w => { if(w.pass) cm[w.word]=(cm[w.word]||0)+1; }));
  const topCorrect = Object.entries(cm).sort((a,b)=>b[1]-a[1]).slice(0,6);

  body.innerHTML = `
    <div class="profile-hero">
      <div class="profile-avatar">${s.name.slice(0,1)}</div>
      <div class="profile-name">${s.name}</div>
      <div style="font-size:13px;opacity:.7;margin-top:2px;">加入日期：${s.created||'-'}</div>
      <div class="profile-stats">
        <div class="profile-stat">
          <div class="profile-stat-num">${avg||'-'}</div>
          <div class="profile-stat-label">平均分數</div>
        </div>
        <div class="profile-stat">
          <div class="profile-stat-num">${recs.length}</div>
          <div class="profile-stat-label">測驗次數</div>
        </div>
        <div class="profile-stat">
          <div class="profile-stat-num">${best||'-'}</div>
          <div class="profile-stat-label">最高分</div>
        </div>
      </div>
    </div>

    ${topWrong.length ? `
    <div class="card">
      <div class="ct">⚠️ 常錯詞語（點擊可加入練習）</div>
      <div class="weak-words">
        ${topWrong.map(([w,c])=>`<div class="weak-chip wc-ng" onclick="practiceWord('${w}')">${w}<span style="font-size:11px;opacity:.6;margin-left:4px;">×${c}</span></div>`).join('')}
      </div>
    </div>` : ''}

    ${topCorrect.length ? `
    <div class="card">
      <div class="ct">✅ 已掌握詞語</div>
      <div class="weak-words">
        ${topCorrect.map(([w])=>`<div class="weak-chip wc-ok">${w}</div>`).join('')}
      </div>
    </div>` : ''}

    <div class="card">
      <div class="ct">📅 測驗歷史紀錄</div>
      ${!recs.length ? '<div style="color:var(--muted);font-size:14px;">還沒有測驗紀錄</div>' :
        recs.slice().reverse().map(r => {
          const sc = r.score>=80?'var(--gd)':r.score>=60?'#a16207':'var(--rd)';
          return `<div class="history-item" style="margin-bottom:8px;">
            <div class="hi-top">
              <div class="hi-date">📅 ${r.date}${r.lesson?'・第'+r.lesson+'課':''}</div>
              <div class="hi-score" style="color:${sc}">${r.score}分</div>
            </div>
            <div class="hi-chips">
              ${(r.results||[]).map(w=>`<div class="hi-chip" style="background:${w.pass?'var(--gl)':'var(--rl)'};color:${w.pass?'var(--gd)':'var(--rd)'}">${w.word}</div>`).join('')}
            </div>
          </div>`;
        }).join('')
      }
    </div>
  `;
}

function practiceWord(word) {
  S.name = S.viewStudent;
  startPrac(word.split(''));
}

function deleteStudent() {
  const name = S.viewStudent;
  if (!name) return;
  if (!confirm(`確定要刪除「${name}」的所有資料嗎？`)) return;
  const arr = getStudents().filter(s => s.name !== name);
  saveStudents(arr);
  renderHomeStudents();
  toast(`已刪除 ${name} 的資料`);
  goPage('pg-students');
  renderStudentManager();
}

// ══════════════════════════════════════════
//  TEST START
// ══════════════════════════════════════════
function goTest() {
  const name = document.getElementById('inp-name').value.trim();
  if (!name) { toast('請先輸入姓名！'); return; }

  // Build word queue from all lessons or show lesson picker
  const db = getLessonsDB();
  const lessons = db[S.pub]?.[S.grade] || {};
  const allWords = Object.values(lessons).flat();
  if (!allWords.length) { toast('請先在老師設定加入詞語！'); return; }

  // Show lesson selector
  showLessonPicker(name, lessons);
}

function showLessonPicker(name, lessons) {
  // Build a simple overlay to pick lessons
  const existing = document.getElementById('lesson-picker');
  if (existing) existing.remove();

  const ov = document.createElement('div');
  ov.id = 'lesson-picker';
  ov.style.cssText = 'position:fixed;inset:0;z-index:500;background:rgba(0,0,0,.5);display:flex;align-items:center;justify-content:center;padding:20px;';

  const lessonEntries = Object.entries(lessons).filter(([,ws])=>ws.length>0);
  ov.innerHTML = `
    <div style="background:#fff;border-radius:24px;padding:24px;width:100%;max-width:400px;box-shadow:0 16px 48px rgba(0,0,0,.2);">
      <div style="font-size:19px;font-weight:900;margin-bottom:4px;">選擇測驗課次</div>
      <div style="font-size:13px;color:var(--muted);margin-bottom:16px;">可多選，或直接全選</div>
      <div style="display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-bottom:16px;" id="lp-grid">
        ${lessonEntries.map(([k,ws])=>`
          <button class="lesson-btn has-words" data-key="${k}"
            onclick="toggleLP(this)" style="position:relative;">
            第${k}課
            <span style="display:block;font-size:10px;opacity:.6;">${ws.length}題</span>
          </button>`).join('')}
      </div>
      <div style="display:flex;gap:8px;margin-bottom:10px;">
        <button class="btn bgh bs" style="flex:1;" onclick="selectAllLP()">全選</button>
        <button class="btn bgh bs" style="flex:1;" onclick="clearAllLP()">清除</button>
      </div>
      <button class="btn bo bl" onclick="startFromPicker('${name}')">開始聽寫 →</button>
      <button class="btn bgh" style="width:100%;margin-top:8px;font-size:15px;padding:10px;" onclick="document.getElementById('lesson-picker').remove()">取消</button>
    </div>`;
  document.body.appendChild(ov);
}

function toggleLP(btn) {
  btn.classList.toggle('on');
}
function selectAllLP() {
  document.querySelectorAll('#lp-grid .lesson-btn').forEach(b => b.classList.add('on'));
}
function clearAllLP() {
  document.querySelectorAll('#lp-grid .lesson-btn').forEach(b => b.classList.remove('on'));
}

function startFromPicker(name) {
  const selected = [...document.querySelectorAll('#lp-grid .lesson-btn.on')].map(b => b.dataset.key);
  if (!selected.length) { toast('請至少選擇一個課次！'); return; }
  document.getElementById('lesson-picker').remove();

  const db = getLessonsDB();
  const lessons = db[S.pub]?.[S.grade] || {};
  let words = [];
  selected.forEach(k => { words = words.concat(lessons[k] || []); });
  // Shuffle
  words = words.sort(() => Math.random() - 0.5);

  S.name = name;
  S.selLesson = selected.join(',');
  S.queue = words;
  S.idx = 0;
  S.results = [];
  upsertStudent(name);
  renderHomeStudents();
  loadQ();
  goPage('pg-dict');
}

function loadQ() {
  document.getElementById('d-qnum').textContent = `第${S.idx+1}題`;
  document.getElementById('d-prog').textContent = `${S.idx+1}/${S.queue.length}`;
  document.getElementById('d-name').textContent = S.name;
  document.getElementById('write-hint').style.display = 'flex';
  clearWrite();
  const pb = document.getElementById('btn-play');
  pb.textContent = '🔊 播放'; pb.disabled = false;
}

// ══════════════════════════════════════════
//  SPEECH
// ══════════════════════════════════════════
function playWord() {
  const w = S.queue[S.idx];
  document.getElementById('write-hint').style.display = 'none';
  const btn = document.getElementById('btn-play');
  btn.textContent = '🔊 播放中...'; btn.disabled = true;
  speechSynthesis.cancel();
  const u = new SpeechSynthesisUtterance(w);
  u.lang = 'zh-TW'; u.rate = S.rate;
  u.onend = u.onerror = () => { btn.textContent = '🔊 再聽'; btn.disabled = false; };
  speechSynthesis.speak(u);
}

function skipWord() {
  const w = S.queue[S.idx];
  S.results.push({ word: w, score: 0, pass: false, detail: '跳過未作答', skipped: true, recognized: '-' });
  toNextOrFinish();
}

// ══════════════════════════════════════════
//  CANVAS
// ══════════════════════════════════════════
let drawing=false, lx=0, ly=0, strokes=[], cur=[];

function initCanvas() {
  const cvs = document.getElementById('write-cvs');
  const wrap = document.getElementById('dict-write');
  resizeCvs(cvs, wrap);
  const pos = (e, c) => { const r=c.getBoundingClientRect(),s=e.touches?e.touches[0]:e; return [s.clientX-r.left, s.clientY-r.top]; };
  const dn = e => { e.preventDefault(); drawing=true; [lx,ly]=pos(e,cvs); cur=[{x:lx,y:ly}]; document.getElementById('write-hint').style.display='none'; };
  const mv = e => { e.preventDefault(); if(!drawing) return; const[x,y]=pos(e,cvs),ctx=cvs.getContext('2d'); ctx.strokeStyle='#1C1917';ctx.lineWidth=8;ctx.lineCap='round';ctx.lineJoin='round';ctx.beginPath();ctx.moveTo(lx,ly);ctx.lineTo(x,y);ctx.stroke();cur.push({x,y});[lx,ly]=[x,y]; };
  const up = () => { if(!drawing) return; drawing=false; if(cur.length>1) strokes.push([...cur]); cur=[]; };
  cvs.addEventListener('mousedown',dn); cvs.addEventListener('mousemove',mv);
  cvs.addEventListener('mouseup',up); cvs.addEventListener('mouseleave',up);
  cvs.addEventListener('touchstart',dn,{passive:false}); cvs.addEventListener('touchmove',mv,{passive:false}); cvs.addEventListener('touchend',up);
}

function resizeCvs(cvs, wrap) {
  const r = wrap.getBoundingClientRect();
  if(r.width<10) return;
  cvs.width=r.width; cvs.height=r.height;
  drawGrid(cvs);
}

function drawGrid(cvs) {
  const ctx=cvs.getContext('2d'),w=cvs.width,h=cvs.height;
  ctx.clearRect(0,0,w,h);
  ctx.strokeStyle='rgba(180,160,140,.3)'; ctx.lineWidth=1; ctx.setLineDash([6,6]);
  ctx.beginPath(); ctx.moveTo(w/2,0); ctx.lineTo(w/2,h); ctx.moveTo(0,h/2); ctx.lineTo(w,h/2); ctx.stroke();
  ctx.setLineDash([]);
}

function clearWrite() {
  strokes=[]; cur=[];
  drawGrid(document.getElementById('write-cvs'));
}

function captureCanvas() {
  const src = document.getElementById('write-cvs');
  const tmp = document.createElement('canvas'); tmp.width=600; tmp.height=600;
  const ctx = tmp.getContext('2d'); ctx.fillStyle='#fff'; ctx.fillRect(0,0,600,600);
  if(!strokes.length) return tmp.toDataURL('image/png').split(',')[1];
  let mnX=Infinity,mnY=Infinity,mxX=-Infinity,mxY=-Infinity;
  strokes.forEach(s=>s.forEach(p=>{mnX=Math.min(mnX,p.x);mnY=Math.min(mnY,p.y);mxX=Math.max(mxX,p.x);mxY=Math.max(mxY,p.y);}));
  const bw=mxX-mnX||1,bh=mxY-mnY||1,pad=50,area=500;
  const sc=Math.min(area/bw,area/bh);
  const ox=pad+(area-bw*sc)/2, oy=pad+(area-bh*sc)/2;
  ctx.strokeStyle='#000'; ctx.lineWidth=Math.max(10,9*sc); ctx.lineCap='round'; ctx.lineJoin='round';
  strokes.forEach(s=>{
    if(s.length<2){ctx.beginPath();ctx.arc(ox+(s[0].x-mnX)*sc,oy+(s[0].y-mnY)*sc,4,0,Math.PI*2);ctx.fill();return;}
    ctx.beginPath(); ctx.moveTo(ox+(s[0].x-mnX)*sc,oy+(s[0].y-mnY)*sc);
    for(let i=1;i<s.length;i++){
      if(i<s.length-1){const mx=(s[i].x+s[i+1].x)/2,my=(s[i].y+s[i+1].y)/2;ctx.quadraticCurveTo(ox+(s[i].x-mnX)*sc,oy+(s[i].y-mnY)*sc,ox+(mx-mnX)*sc,oy+(my-mnY)*sc);}
      else{ctx.lineTo(ox+(s[i].x-mnX)*sc,oy+(s[i].y-mnY)*sc);}
    }
    ctx.stroke();
  });
  return tmp.toDataURL('image/png').split(',')[1];
}

// ══════════════════════════════════════════
//  SUBMIT & SCORE
// ══════════════════════════════════════════
async function submitAnswer() {
  if(!strokes.length){ toast('請先寫字再送出！'); return; }
  const word = S.queue[S.idx];
  goPage('pg-score');
  const img = captureCanvas();
  const r = await ocrScore(word, img);
  S.results.push(r);
  S.lastResult = r;
  showResult(r);
}

async function ocrScore(word, img) {
  try {
    const resp = await fetch(`${S.server}/ocr`, {
      method:'POST', headers:{'Content-Type':'application/json'},
      body:JSON.stringify({image:img,expected:word,mode:'score',stroke_count:strokes.length}),
      signal:AbortSignal.timeout(20000)
    });
    if(!resp.ok) throw new Error(`HTTP ${resp.status}`);
    const d = await resp.json();
    const score = typeof d.score==='number'?d.score:(d.correct?100:0);
    return {
      word, recognized:d.recognized||'?', correct:d.correct===true,
      score, pass:score>=60,
      detail:d.detail||(d.correct?'寫對了！':`辨識為「${d.recognized||'?'}」`),
      stroke_order:d.stroke_order_ok, clarity:d.clarity_ok, skipped:false
    };
  } catch(e) {
    return {word,recognized:'?',correct:false,score:0,pass:false,
      detail:'⚠️ 無法連線 OCR 伺服器，請確認 ocr_server.py 已啟動',skipped:false,error:true};
  }
}

function showResult(r) {
  const card = document.getElementById('result-card');
  const isLast = S.idx >= S.queue.length-1;
  const barC = r.score>=80?'var(--green)':r.score>=60?'var(--yellow)':'var(--red)';
  const bg = r.pass?'var(--gl)':'var(--rl)';
  const col = r.pass?'var(--green)':'var(--red)';

  const bkRows = [];
  if(!r.skipped&&!r.error){
    bkRows.push(['詞語辨識',r.correct?'✓ 正確':`✗ 辨識「${r.recognized}」`,r.correct]);
    if(r.stroke_order!==undefined) bkRows.push(['筆畫順序',r.stroke_order?'✓ 正確':'✗ 需改善',r.stroke_order]);
    if(r.clarity!==undefined)      bkRows.push(['字跡清晰',r.clarity?'✓ 清晰':'△ 稍潦草',r.clarity]);
  }

  const showPrac = !r.pass && !r.skipped;

  card.innerHTML = `
    <div class="verdict" style="background:${bg}">
      <div class="vi">${r.skipped?'⏭️':r.pass?'🎉':'😅'}</div>
      <div style="flex:1">
        <div class="vword" style="color:${col}">${r.word}</div>
        <div class="vmsg" style="color:${r.pass?'var(--gd)':'var(--rd)'}">${r.detail}</div>
      </div>
    </div>
    ${!r.skipped&&!r.error?`
    <div>
      <div style="display:flex;justify-content:space-between;margin-bottom:5px;font-size:14px;font-weight:700;">
        <span style="color:var(--muted)">得分</span>
        <span style="color:${barC}">${r.score}分</span>
      </div>
      <div class="bar-wrap"><div class="bar-fill" style="width:${r.score}%;background:${barC}"></div></div>
    </div>
    ${bkRows.length?`<div style="display:flex;flex-direction:column;gap:7px;">
      ${bkRows.map(([l,v,ok])=>`<div class="bk-row"><span class="bk-l">${l}</span>
      <span class="bk-v" style="color:${ok?'var(--gd)':'var(--rd)'}">${v}</span></div>`).join('')}
    </div>`:''}` : ''}
    <div class="res-acts">
      ${showPrac?`<button class="btn bg bl" id="prac-btn">✍️ 練習寫「${r.word}」</button>`:''}
      <button class="btn bo bl" id="next-btn">${isLast?'🎉 完成測驗':'下一題 →'}</button>
    </div>`;

  // 用 getElementById 綁定事件，避免 onclick 字串裡的引號問題
  const pracBtn = document.getElementById('prac-btn');
  if(pracBtn){
    pracBtn.addEventListener('click', function(){
      startPrac(r.word.split(''));
    });
  }
  const nextBtn = document.getElementById('next-btn');
  if(nextBtn){
    nextBtn.addEventListener('click', function(){
      if(isLast) finishTest(); else nextQ();
    });
  }

  goPage('pg-result');
}

function nextQ() { S.idx++; loadQ(); goPage('pg-dict'); }
function toNextOrFinish() { if(S.idx>=S.queue.length-1) finishTest(); else nextQ(); }

// ══════════════════════════════════════════
//  PRACTICE
// ══════════════════════════════════════════
let pracD=false,pLx=0,pLy=0,pracS=[],pracC=[];

function startPrac(chars) {
  S.pracQueue=chars; S.pracIdx=0;
  loadPracChar(); goPage('pg-practice');
}

function loadPracChar() {
  const ch = S.pracQueue[S.pracIdx];
  document.getElementById('prac-lbl').textContent = `${S.pracIdx+1}/${S.pracQueue.length}：「${ch}」`;
  document.getElementById('prac-chars-row').innerHTML = S.pracQueue.map((c,i)=>
    `<div class="pchar ${i===S.pracIdx?'cur':i<S.pracIdx?'done':''}">${c}</div>`).join('');

  // 正確字展示（讓孩子看清楚要寫什麼）
  const ansEl = document.getElementById('prac-answer-display');
  if(ansEl) ansEl.textContent = S.pracQueue.join('　');

  // 把正確字渲染到描寫底圖
  const ghost = document.getElementById('prac-ghost');
  ghost.textContent = ch;

  clearPrac();
  // 重新繪製田字格
  setTimeout(() => {
    const cvs = document.getElementById('prac-cvs');
    const wrap = document.getElementById('prac-write');
    const rect = wrap.getBoundingClientRect();
    cvs.width = rect.width;
    cvs.height = rect.height;
    drawPracGrid(cvs);
  }, 50);
}

function drawPracGrid(cvs) {
  const ctx = cvs.getContext('2d');
  const w = cvs.width, h = cvs.height;
  ctx.clearRect(0, 0, w, h);
  // 田字格
  ctx.strokeStyle = 'rgba(249,115,22,.2)';
  ctx.lineWidth = 1.5;
  ctx.setLineDash([6, 6]);
  ctx.beginPath();
  ctx.moveTo(w/2, 0); ctx.lineTo(w/2, h);
  ctx.moveTo(0, h/2); ctx.lineTo(w, h/2);
  ctx.stroke();
  ctx.setLineDash([]);
  // 外框
  ctx.strokeStyle = 'rgba(249,115,22,.15)';
  ctx.lineWidth = 2;
  ctx.strokeRect(2, 2, w-4, h-4);
}

function initPracCanvas() {
  const cvs = document.getElementById('prac-cvs');
  const wrap = document.getElementById('prac-write');
  const rz=()=>{const r=wrap.getBoundingClientRect();cvs.width=r.width;cvs.height=r.height;};
  rz();
  const pos=e=>{const r=cvs.getBoundingClientRect(),s=e.touches?e.touches[0]:e;return[s.clientX-r.left,s.clientY-r.top];};
  cvs.addEventListener('mousedown',e=>{pracD=true;[pLx,pLy]=pos(e);pracC=[{x:pLx,y:pLy}];});
  cvs.addEventListener('mousemove',e=>{
    if(!pracD)return;const[x,y]=pos(e),ctx=cvs.getContext('2d');
    ctx.strokeStyle='rgba(249,115,22,.85)';ctx.lineWidth=8;ctx.lineCap='round';ctx.lineJoin='round';
    ctx.beginPath();ctx.moveTo(pLx,pLy);ctx.lineTo(x,y);ctx.stroke();pracC.push({x,y});[pLx,pLy]=[x,y];
  });
  const up2=()=>{pracD=false;if(pracC.length>1)pracS.push([...pracC]);pracC=[];};
  cvs.addEventListener('mouseup',up2);cvs.addEventListener('mouseleave',up2);
  cvs.addEventListener('touchstart',e=>{e.preventDefault();pracD=true;[pLx,pLy]=pos(e);pracC=[{x:pLx,y:pLy}];},{passive:false});
  cvs.addEventListener('touchmove',e=>{
    e.preventDefault();if(!pracD)return;const[x,y]=pos(e),ctx=cvs.getContext('2d');
    ctx.strokeStyle='rgba(249,115,22,.85)';ctx.lineWidth=8;ctx.lineCap='round';ctx.lineJoin='round';
    ctx.beginPath();ctx.moveTo(pLx,pLy);ctx.lineTo(x,y);ctx.stroke();pracC.push({x,y});[pLx,pLy]=[x,y];
  },{passive:false});
  cvs.addEventListener('touchend',up2);
}

function clearPrac() {
  pracS=[];pracC=[];
  const cvs=document.getElementById('prac-cvs');
  cvs.getContext('2d').clearRect(0,0,cvs.width,cvs.height);
}

function nextPrac() {
  S.pracIdx++;
  if(S.pracIdx>=S.pracQueue.length){
    toast('✓ 補強練習完成！');
    if(S.lastResult) showResult(S.lastResult);
    else goPage('pg-result');
  } else loadPracChar();
}

// ══════════════════════════════════════════
//  FINISH
// ══════════════════════════════════════════
function finishTest() {
  const total = S.results.length ? Math.round(S.results.reduce((a,r)=>a+r.score,0)/S.results.length) : 0;

  // Save to student record
  addRecord(S.name, {
    date: new Date().toLocaleString('zh-TW'),
    lesson: S.selLesson,
    score: total,
    results: S.results.map(r=>({word:r.word,score:r.score,pass:r.pass}))
  });

  document.getElementById('final-name').textContent = `${S.name} 的成績`;
  document.getElementById('final-score').textContent = total;

  const g = total>=90?{t:'太棒了！',bg:'var(--gl)',c:'var(--gd)'}
           :total>=75?{t:'很好！',  bg:'var(--gl)',c:'var(--gd)'}
           :total>=60?{t:'加油！',  bg:'var(--yl)',c:'#a16207'}
                     :{t:'繼續練習',bg:'var(--rl)',c:'var(--rd)'};
  const gb = document.getElementById('final-grade');
  gb.textContent=g.t; gb.style.background=g.bg; gb.style.color=g.c;

  document.getElementById('final-list').innerHTML = S.results.map(r=>{
    const sc=r.score>=80?'var(--gd)':r.score>=60?'#a16207':'var(--rd)';
    return `<div class="wri-item"><div class="wri-ic">${r.skipped?'⏭️':r.pass?'✅':'❌'}</div>
      <div class="wri-word">${r.word}</div>
      <div class="wri-sc" style="color:${sc}">${r.score}分</div></div>`;
  }).join('');

  const wrongs = S.results.filter(r=>!r.pass&&!r.skipped);
  document.getElementById('btn-retry').style.display = wrongs.length?'':'none';
  goPage('pg-final');
}

function retryWrong() {
  const ws = S.results.filter(r=>!r.pass&&!r.skipped).map(r=>r.word);
  if(!ws.length){toast('沒有需要重考的詞語！');return;}
  S.queue=ws; S.idx=0; S.results=[];
  loadQ(); goPage('pg-dict');
}

// ══════════════════════════════════════════
//  INIT
// ══════════════════════════════════════════
window.addEventListener('load', () => {
  const sv = localStorage.getItem('ocr_server');
  if(sv) S.server = sv;

  initCanvas();
  initPracCanvas();
  renderHomeStudents();

  new MutationObserver(()=>{
    if(document.getElementById('pg-dict').classList.contains('on')){
      setTimeout(()=>resizeCvs(document.getElementById('write-cvs'),document.getElementById('dict-write')),60);
    }
  }).observe(document.getElementById('pg-dict'),{attributes:true,attributeFilter:['class']});

  window.addEventListener('resize',()=>{
    resizeCvs(document.getElementById('write-cvs'),document.getElementById('dict-write'));
    const pc=document.getElementById('prac-cvs'),pw=document.getElementById('prac-write');
    const r=pw.getBoundingClientRect(); pc.width=r.width; pc.height=r.height;
  });
});
</script>
</body>
</html>

import os
os.environ['PADDLE_PDX_DISABLE_MODEL_SOURCE_CHECK'] = 'True'

import base64
import re
from io import BytesIO

import numpy as np
from PIL import Image, ImageEnhance
from flask import Flask, request, jsonify
from flask_cors import CORS
from paddleocr import PaddleOCR

app = Flask(__name__)
CORS(app)

print("正在初始化 OCR 模型（請稍候）...")
ocr = PaddleOCR(
    lang='chinese_cht',
    det_db_thresh=0.2,
    det_db_box_thresh=0.4,
    det_db_unclip_ratio=2.0,
    rec_batch_num=1,
    use_angle_cls=False,
    enable_mkldnn=False,
    cpu_threads=1,
    show_log=False,
)
print("✅ OCR 模型載入完成！")


def cjk_only(text):
    return re.sub(r'[^\u4e00-\u9fff\u3400-\u4dbf]', '', text)


def b64_to_np(b64_str):
    if ',' in b64_str:
        b64_str = b64_str.split(',')[1]
    img = Image.open(BytesIO(base64.b64decode(b64_str))).convert('RGB')
    return np.array(img)


def preprocess_image(img_np):
    img = Image.fromarray(img_np).convert('RGB')
    img = ImageEnhance.Contrast(img).enhance(1.8)
    w, h = img.size
    padded = Image.new('RGB', (w + 60, h + 60), (255, 255, 255))
    padded.paste(img, (30, 30))
    return np.array(padded)


def ocr_single(img_np):
    """對單張圖片執行 OCR，嘗試多種格式繞開 primitive bug"""
    h, w = img_np.shape[:2]
    if h < 64 or w < 64:
        img = Image.fromarray(img_np)
        padded = Image.new('RGB', (max(w, 64), max(h, 64)), (255, 255, 255))
        padded.paste(img, (0, 0))
        img_np = np.array(padded)

    # 嘗試策略：原圖、灰階轉回RGB、不同長寬比
    candidates = []

    # 策略1: 原圖不同尺寸
    for sw, sh in [(800,400),(640,320),(960,480),(512,256),(400,200)]:
        arr = np.array(Image.fromarray(img_np).resize((sw, sh), Image.LANCZOS))
        candidates.append(arr)

    # 策略2: 灰階轉回 RGB（改變記憶體排列）
    for sw, sh in [(800,400),(640,320),(960,480)]:
        arr = np.array(
            Image.fromarray(img_np).convert('L').convert('RGB').resize((sw, sh), Image.LANCZOS)
        )
        candidates.append(arr)

    # 策略3: 正方形
    for sz in [400, 512, 300]:
        arr = np.array(Image.fromarray(img_np).resize((sz, sz), Image.LANCZOS))
        candidates.append(arr)

    for arr in candidates:
        try:
            result = ocr.ocr(arr, cls=False)
            all_text = ''
            max_conf = 0.0
            if result and result[0]:
                for line in result[0]:
                    if line and len(line) >= 2:
                        txt, conf = line[1]
                        all_text += txt
                        if conf > max_conf:
                            max_conf = conf
            if all_text:
                return all_text, max_conf
        except Exception as e:
            if 'primitive' in str(e).lower() or 'execute' in str(e).lower():
                continue
            print(f"[DEBUG] ocr error: {e}")
            return '', 0.0
    return '', 0.0


def run_ocr(img_np):
    """整張 + 多種切割，智慧合併結果"""
    img_np = preprocess_image(img_np)
    h, w = img_np.shape[:2]

    # 1. 整張圖辨識
    t, c = ocr_single(img_np)
    whole_cjk = cjk_only(t)
    print(f"[DEBUG] whole: cjk='{whole_cjk}' conf={c:.2f}")

    # 如果整張辨識信心度高，直接用
    if whole_cjk and c >= 0.80:
        return whole_cjk, c

    # 2. 切成左右兩半，每半只取「最高信心度的單一字」
    collected = list(whole_cjk)  # 從整張結果開始
    collected_conf = c

    for ratio in [0.5, 0.4, 0.6]:
        split = int(w * ratio)
        for i, region in enumerate([img_np[:, :split], img_np[:, split:]]):
            img = Image.fromarray(region)
            padded = Image.new('RGB', (img.width + 60, img.height + 40), (255, 255, 255))
            padded.paste(img, (30, 20))
            t2, c2 = ocr_single(np.array(padded))
            cjk_t2 = cjk_only(t2)

            # 只接受長度 1~2 且信心度 >= 0.6 的結果
            if cjk_t2 and c2 >= 0.60 and len(cjk_t2) <= 2:
                print(f"[DEBUG] split{int(ratio*100)}[{i}]: cjk='{cjk_t2}' conf={c2:.2f}")
                for ch in cjk_t2:
                    if ch not in collected:
                        collected.append(ch)
                        if c2 > collected_conf:
                            collected_conf = c2

    result = ''.join(collected)
    print(f"[DEBUG] merged: '{result}'")
    return result, collected_conf


@app.route('/health')
def health():
    return jsonify({'status': 'ok', 'engine': 'PaddleOCR', 'lang': 'chinese_cht'})


@app.route('/ocr', methods=['POST'])
def do_ocr():
    try:
        data = request.get_json()
        if not data or 'image' not in data:
            return jsonify({'error': '缺少 image 欄位'}), 400

        img_np   = b64_to_np(data['image'])
        expected = data.get('expected', '')
        mode     = data.get('mode', 'score')

        all_text, max_conf = run_ocr(img_np)
        cjk_text = cjk_only(all_text)

        print(f"[OCR] mode={mode} expected='{expected}' recognized='{cjk_text}' conf={max_conf:.2f}")

        if mode == 'wordlist':
            return jsonify({'raw': all_text.strip(), 'cjk': cjk_text, 'confidence': round(max_conf, 3)})

        if not cjk_text:
            return jsonify({'recognized': '?', 'correct': False, 'score': 0,
                           'detail': '字跡無法辨識，請寫清楚一點', 'confidence': 0.0})

        if expected:
            chars_found   = [c for c in expected if c in cjk_text]
            chars_missing = [c for c in expected if c not in cjk_text]
            score   = round(len(chars_found) / len(expected) * 100)
            correct = len(chars_missing) == 0
            if correct:
                detail = '寫對了！'
            elif chars_found:
                detail = f'找到「{"".join(chars_found)}」，但缺少「{"".join(chars_missing)}」'
            else:
                detail = f'辨識為「{cjk_text}」，正確應為「{expected}」'
        else:
            correct, score, detail = True, 100, '完成'

        print(f"[OCR] expected='{expected}' cjk='{cjk_text}' found={chars_found if expected else []} score={score}")

        return jsonify({'recognized': cjk_text, 'correct': correct, 'score': score,
                       'detail': detail, 'confidence': round(max_conf, 3)})

    except Exception as e:
        import traceback
        traceback.print_exc()
        return jsonify({'error': str(e)}), 500


if __name__ == '__main__':
    import os
    port = int(os.environ.get('PORT', 5000))
    print("\n" + "=" * 45)
    print("  國語聽寫系統 OCR 伺服器")
    print(f"  網址：http://0.0.0.0:{port}")
    print("=" * 45 + "\n")
    app.run(host='0.0.0.0', port=port, debug=False)
