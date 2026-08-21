// ==UserScript==
// @name         TritoX AgencyZoom Auto-Fill
// @namespace    http://tampermonkey.net/
// @version      3.0
// @description  Auto-fill AgencyZoom fields from TritoX QC data
// @match        https://app.agencyzoom.com/*
// @match        https://gnanaamoorthy-coder.github.io/*
// @grant        GM_setValue
// @grant        GM_getValue
// @grant        GM_addStyle
// ==/UserScript==

(function(){
  'use strict';

  console.log('[TritoX TM] hostname:', window.location.hostname);
  if(window.location.hostname === 'gnanaamoorthy-coder.github.io'){
    console.log('[TritoX TM] Running on TritoX page');
    function checkTritoXData(){
      try{
        const raw = localStorage.getItem('tritox_az_data');
        if(!raw) return;
        const data = JSON.parse(raw);
        if(!data || !data._name || !data._ts) return;
        const gmRaw = GM_getValue('tritox_az_data','');
        let gmTs = 0;
        try{ gmTs = JSON.parse(gmRaw)._ts || 0; }catch(e){}
        if(data._ts > gmTs){
          GM_setValue('tritox_az_data', JSON.stringify(data));
          console.log('[TritoX TM] Saved to GM for:', data._name);
        }
      }catch(e){ console.log('[TritoX TM] Error:', e); }
    }
    setInterval(checkTritoXData, 1000);
    return;
  }

  console.log('[TritoX TM] Running on AgencyZoom page');
  function addFillButton(){
    const vehicleField = document.getElementById('customfields-cf30203');
    if(!vehicleField) return;
    if(document.getElementById('tritox-fill-btn')) return;
    const raw = GM_getValue('tritox_az_data','');
    if(!raw) return;
    let data;
    try{ data = JSON.parse(raw); }catch(e){ return; }
    if(!data || !data._name) return;
    const age = Date.now() - (data._ts || 0);
    if(age > 7200000) return;
    const btn = document.createElement('div');
    btn.id = 'tritox-fill-btn';
    btn.style.cssText = 'position:fixed;top:80px;right:20px;z-index:99999;background:linear-gradient(135deg,#00d4ff,#7b2fff);color:#fff;padding:10px 16px;border-radius:10px;cursor:pointer;font-size:13px;font-weight:700;box-shadow:0 4px 20px rgba(0,212,255,0.4);font-family:sans-serif;text-align:center;min-width:160px;';
    btn.innerHTML = '🚀 Fill from TritoX<br><span style="font-size:11px;font-weight:400;opacity:0.9;">' + data._name + '</span>';
    btn.addEventListener('click', function(){
      // Show confirmation popup
      const overlay = document.createElement('div');
      overlay.style.cssText = 'position:fixed;inset:0;background:rgba(0,0,0,0.7);z-index:999999;display:flex;align-items:center;justify-content:center;';
      const box = document.createElement('div');
      box.style.cssText = 'background:#fff;border-radius:16px;padding:28px 32px;max-width:380px;width:90%;text-align:center;font-family:sans-serif;box-shadow:0 20px 60px rgba(0,0,0,0.3);';
      box.innerHTML = '<div style="font-size:32px;margin-bottom:12px;">⚠️</div>'
        +'<div style="font-size:16px;font-weight:700;color:#1a1a2e;margin-bottom:8px;">Confirm Auto-Fill</div>'
        +'<div style="font-size:13px;color:#666;margin-bottom:6px;">You are about to fill this lead with:</div>'
        +'<div style="font-size:15px;font-weight:700;color:#7b2fff;margin-bottom:20px;padding:10px;background:#f0e8ff;border-radius:8px;">'+data._name+'</div>'
        +'<div style="font-size:12px;color:#999;margin-bottom:20px;">Make sure you are on the correct lead in AgencyZoom before confirming.</div>'
        +'<div style="display:flex;gap:10px;justify-content:center;">'
        +'<button id="tritox-cancel" style="flex:1;padding:10px;border:2px solid #ddd;background:#fff;border-radius:8px;cursor:pointer;font-size:13px;font-weight:600;color:#666;">❌ Cancel</button>'
        +'<button id="tritox-confirm" style="flex:1;padding:10px;border:none;background:linear-gradient(135deg,#00d4ff,#7b2fff);border-radius:8px;cursor:pointer;font-size:13px;font-weight:700;color:#fff;">✅ Yes, Fill!</button>'
        +'</div>';
      overlay.appendChild(box);
      document.body.appendChild(overlay);

      // Cancel
      document.getElementById('tritox-cancel').addEventListener('click', function(){
        overlay.remove();
      });

      // Confirm
      document.getElementById('tritox-confirm').addEventListener('click', function(){
        overlay.remove();
        fillFields(data);
        // Store timestamp of this fill to prevent reappearing
        try{
          const d = JSON.parse(GM_getValue('tritox_az_data','{}'));
          filledTs = d._ts || Date.now();
        }catch(e){ filledTs = Date.now(); }
        GM_setValue('tritox_az_data','');

        // Show done state with close button and countdown
        let secs = 8;
        btn.style.background = 'linear-gradient(135deg,#00e887,#00b359)';
        btn.style.minWidth = '180px';

        function updateBtn(){
          btn.innerHTML = '✅ Done! Review & Save<br>'
            +'<span style="font-size:11px;font-weight:400;opacity:0.9;">' + data._name + '</span><br>'
            +'<div style="display:flex;align-items:center;justify-content:center;gap:8px;margin-top:6px;">'
            +'<span style="font-size:10px;opacity:0.8;">Auto close in '+secs+'s</span>'
            +'<button id="tritox-close-btn" style="background:rgba(255,255,255,0.25);border:1px solid rgba(255,255,255,0.5);color:#fff;border-radius:6px;padding:2px 8px;cursor:pointer;font-size:11px;font-weight:700;">✕ Close</button>'
            +'</div>';

          // Attach close button listener after innerHTML update
          const closeBtn = document.getElementById('tritox-close-btn');
          if(closeBtn){
            closeBtn.addEventListener('click', function(e){
              e.stopPropagation();
              btn.remove();
            });
          }
        }

        updateBtn();

        // Countdown timer
        const timer = setInterval(function(){
          secs--;
          if(secs <= 0){
            clearInterval(timer);
            btn.remove();
          } else {
            updateBtn();
          }
        }, 1000);
      });

      // Click outside to cancel
      overlay.addEventListener('click', function(e){
        if(e.target === overlay) overlay.remove();
      });
    });
    document.body.appendChild(btn);
  }

  function fillText(id, val){
    if(!val && val !== 0) return;
    const el = document.getElementById('customfields-' + id);
    if(!el) return;
    try{
      el.focus();
      const setter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype,'value').set;
      setter.call(el, String(val));
      el.dispatchEvent(new Event('focus',{bubbles:true}));
      el.dispatchEvent(new Event('input',{bubbles:true}));
      el.dispatchEvent(new Event('change',{bubbles:true}));
      el.dispatchEvent(new KeyboardEvent('keydown',{bubbles:true}));
      el.dispatchEvent(new KeyboardEvent('keyup',{bubbles:true}));
      el.blur();
      el.dispatchEvent(new Event('blur',{bubbles:true}));
    }catch(e){}
  }

  function fillSelect(id, val){
    if(!val) return;
    const el = document.getElementById('customfields-' + id);
    if(!el) return;
    try{
      const setter = Object.getOwnPropertyDescriptor(window.HTMLSelectElement.prototype,'value').set;
      setter.call(el, String(val));
      el.dispatchEvent(new Event('change',{bubbles:true}));
    }catch(e){}
  }

  function fillFields(d){
    fillText('cf30203', d.vehicles_policy);
    fillSelect('cf56698', d.bodily_injury);
    fillText('cf47028', d.home_coverage_a);
    fillText('cf37981', d.home_annual);
    fillText('cf56654', d.auto1);
    fillSelect('cf56655', d.auto1_ded);
    fillText('cf56656', d.auto2);
    fillSelect('cf56657', d.auto2_ded);
    fillText('cf56692', d.auto3);
    fillSelect('cf56693', d.auto3_ded);
    fillText('cf56694', d.auto4);
    fillSelect('cf56695', d.auto4_ded);
    fillText('cf56696', d.auto5);
    fillSelect('cf56697', d.auto5_ded);
    function fillMoneyFields(attempt){
      const mEl = document.querySelector('input[name="customFields[cf30197]"]');
      const sEl = document.querySelector('input[name="customFields[cf30199]"]');
      if(mEl){
        const setter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype,'value').set;
        setter.call(mEl, String(d.monthly_auto).replace(/[$,]/g,''));
        mEl.dispatchEvent(new Event('input',{bubbles:true}));
        mEl.dispatchEvent(new Event('change',{bubbles:true}));
      }
      if(sEl){
        const setter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype,'value').set;
        setter.call(sEl, String(d.auto_6months).replace(/[$,]/g,''));
        sEl.dispatchEvent(new Event('input',{bubbles:true}));
        sEl.dispatchEvent(new Event('change',{bubbles:true}));
      }
      if((!mEl || !sEl) && attempt < 30){
        setTimeout(function(){ fillMoneyFields(attempt+1); }, 800);
      }
    }
    fillMoneyFields(1);
  }

  let lastCheck = '';
  let filledTs = 0; // timestamp of last fill action
  
  setInterval(function(){
    // Get current GM data
    const gmRaw = GM_getValue('tritox_az_data','');
    if(!gmRaw){
      // No data — button should not show
      return;
    }
    
    let gmData;
    try{ gmData = JSON.parse(gmRaw); }catch(e){ return; }
    
    // If GM data is newer than last fill → new PDF processed → show button
    const gmTs = gmData._ts || 0;
    if(gmTs <= filledTs){
      // Same or older data — already filled this PDF
      return;
    }
    
    // New PDF data available — show button
    const cur = window.location.href + (document.querySelector('.az-form-group') ? '1' : '0');
    if(cur !== lastCheck){
      lastCheck = cur;
      const existing = document.getElementById('tritox-fill-btn');
      if(existing) existing.remove();
      setTimeout(addFillButton, 1200);
    }
    addFillButton();
  }, 1500);

  setTimeout(addFillButton, 2000);

})();
