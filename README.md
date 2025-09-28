
<html>
<head>
	<meta charset="utf-8">
	<title>Time Cipher</title>
	<style>
		body { font-family: system-ui, -apple-system, "Segoe UI", Roboto, Arial; padding: 2rem; background:#111; color:#eee; }
		.card { background:#151515; padding:1.5rem; border-radius:8px; max-width:540px; margin:0 auto; box-shadow:0 6px 20px rgba(0,0,0,.6); }
		.lock { color:#f66; }
		.open { color:#6f6; font-weight:600; font-size:1.25rem; }
		.small { font-size:0.9rem; color:#aaa; margin-top:0.5rem; }
	</style>
</head>
<body>
	<div class="card">
		<h2>Secure Message</h2>
		<div id="output" class="lock">Locked</div>
		<div id="countdown" class="small"></div>
	</div>
	<script>
		// lightweight obfuscation: reversed base64 and reversed key codes
		(function(){
			// reversed base64 of the XOR-encrypted payload
			var revB64 = "==gFSwQVURBZD8VI"; // reversed "IV8ZDBRUVQwSFg=="
			// key char codes reversed: original key is "t1mex" (chars: 116,49,109,101,120)
			var revKeyCodes = [120,101,109,49,116].reverse(); // further reverse here to avoid visual key
			var encryptedB64 = revB64.split('').reverse().join('');
			var key = String.fromCharCode.apply(null, revKeyCodes); // yields the key
			function decryptXorFromB64(b64, keyStr){
				try {
					var raw = atob(b64);
				} catch(e) {
					return null;
				}
				var out = [], klen = keyStr.length;
				for(var i=0;i<raw.length;i++){
					out.push(String.fromCharCode(raw.charCodeAt(i) ^ keyStr.charCodeAt(i % klen)));
				}
				return out.join('');
			}
			var outEl = document.getElementById('output');
			var cdEl = document.getElementById('countdown');
			function showMessage(msg){
				outEl.textContent = msg;
				outEl.className = "open";
			}
			function showLocked(){
				outEl.textContent = "Locked — available only between 3:00 and 3:59 AM";
				outEl.className = "lock";
			}
			function update(){
				var now = new Date();
				var hour = now.getHours();
				if(hour === 3){
					// reveal decrypted message
					var plain = decryptXorFromB64(encryptedB64, key);
					if(plain) showMessage(plain);
					else showLocked();
					cdEl.textContent = "This message is visible now.";
				} else {
					showLocked();
					var next = new Date(now.getTime());
					next.setHours(3,0,0,0);
					if(now.getHours() > 3 || (now.getHours()===3 && now.getMinutes()>59)) {
						// if past today's 3am, set to tomorrow
						next.setDate(next.getDate() + 1);
					} else if(now.getHours() < 3){
						// earlier today at 3am later same day is fine
					}
					var diff = Math.max(0, next - now);
					var hrs = Math.floor(diff / 3600000);
					var mins = Math.floor((diff % 3600000) / 60000);
					var secs = Math.floor((diff % 60000) / 1000);
					cdEl.textContent = "Next reveal in " + hrs + "h " + mins + "m " + secs + "s.";
				}
			}
			update();
			setInterval(update, 1000);
			setTimeout(function(){
				try {
					revB64 = null;
					revKeyCodes = null;
					encryptedB64 = null;
				} catch(e){}
			}, 2000);
		})();
	</script>
</body>
</html>
