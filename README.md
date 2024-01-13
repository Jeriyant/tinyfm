# Install File Manager Tinyfm di OpenWRT
opkg update && opkg install luci-app-uhttpd php8 php8-cgi tar php8 php8-cgi php8-mod-session php8-mod-ctype php8-mod-fileinfo php8-mod-mbstring iconv php8-mod-ctype php8-mod-fileinfo php8-mod-iconv php8-mod-json php8-mod-mbstring php8-mod-session php8-mod-zip

# Agar Browser Tidak download file php, kita harus config Service > uHTTPd terlebih dahulu, silakan eksekusi perintah berikut ini di SSH OpenWrt :

if ! grep -q ".php=/usr/bin/php-cgi" /etc/config/uhttpd; then
	uci set uhttpd.main.ubus_prefix='/ubus'
	uci set uhttpd.main.interpreter='.php=/usr/bin/php-cgi'
	uci set uhttpd.main.index_page='cgi-bin/luci'
	uci add_list uhttpd.main.index_page='index.html'
	uci add_list uhttpd.main.index_page='index.php'
	uci commit uhttpd
	/etc/init.d/uhttpd restart
fi

# Download file Tinyfm ke Folder /www
mkdir /www/jr222articvbejlm
cd /www/jr222articvbejlm
wget https://raw.githubusercontent.com/Jeriyant/tinyfm/main/tinyfm.zip

# Install Aplikasi Unzip, Extract file tinyfm.zip dan Config Agar File Manager Bisa Membaca Semua Folder

opkg install unzip
cd /www/jr222articvbejlm
unzip tinyfm.zip && rm tinyfm.zip
cd tinyfm && ln -s / rootfs

# Menambah Menu Tiny File Manager di WebUI OpenWrt
cat <<'EOF' >/usr/lib/lua/luci/controller/tinyfm.lua
module("luci.controller.tinyfm", package.seeall)
function index()
entry({"admin","services","tinyfm"}, template("tinyfm"), _("File Manager"), 45).leaf=true
end
EOF

cat <<'EOF' >/usr/lib/lua/luci/view/tinyfm.htm
<%+header%>
<div class="cbi-map"><br>
<iframe id="tinyfm" style="width: 100%; min-height: 100vh; border: none; border-radius: 2px;"></iframe>
</div>
<script type="text/javascript">
document.getElementById("tinyfm").src = window.location.protocol + "//" + window.location.host + "/jr222articvbejlm/tinyfm/tinyfm.php";
</script>
<%+footer%>
EOF

