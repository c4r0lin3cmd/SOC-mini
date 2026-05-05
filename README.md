# SOC-mini
Implementasi Wazuh SIEM, Web Application Firewall, dan Tools Monitoring untuk Deteksi Serangan pada Arsitektur Virtual Multi-VM (Kali Linux sebagai Attacker dan Ubuntu Server sebagai Target)

# Arsitektur
<img width="785" height="859" alt="image" src="https://github.com/user-attachments/assets/1c7afe54-a326-468b-bd35-af8b07d5b8aa" />



# Fitur Utama
1. Integrasi SIEM Terpusat (Wazuh)
2. Simulasi Lingkungan Serangan Nyata
3. Implementasi Web Application Firewall (WAF)
4. Deteksi Intrusi Berbasis Log dan Rule
5. Monitoring dan Alerting Real-Time
6. Isolasi Lingkungan Uji (Virtual Lab)
7. Ekstensibilitas Tools Keamanan

# Note 
1. Server SIEM : Server untuk SIEM manajer
2. Server Target : Server yang akan digunakan untuk Deploy Website, Install WAF ModSecurity, Wazuh Agent, dan Suricata IDS
3. Server Attacker : Server yang digunakan untuk menyerang 

# Installasi Server SIEM
1. Buka VM > New > Isi Vm Name, VM Folder sesuaikan dengan penyimpanan yang diinginkan > Select ISO Image dengan mengklik tanda panah kebawah dan pilih iso ubuntu server yang sudah di download.
   <img width="946" height="761" alt="image" src="https://github.com/user-attachments/assets/1d0462a2-bbc1-4885-99ad-1a979516014a" />

3. Atur username, password, serta hostname yang diinginkan
4. Atur 'Specify virtual hardware' dengan spesifikasi sebagai berikut
   Base Memory : 4GB
   Processors  : 2 core
   > Klik Finish
5. Klik Setting > Network buka di tab Expert > Pastikan pada Adapter 1 attachednya 'Bridged Adapter'
   <img width="1161" height="757" alt="image" src="https://github.com/user-attachments/assets/3af46617-e9cf-4cde-9a4e-1344c91cfd86" />
6. Klik Start
7. Update dan Upgrade package
   ```
   sudo apt update && sudo apt upgrade -y
   ```
8. install wazuh langsung dari sumber resimnya
   ```
   curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
   ```
9. Verifikasi installasi wazuh dengan perintah
   ```
   sudo systemctl status wazuh-manager
   ```
   Jika status active (running) maka wazuh sudah siap digunakan.
10. Pada terminal ubuntu server, jalankan perintah
    ```
    ip a
    ```
    Lihat pada enp0 > inet itu adalah alamat ip ubuntu servernya
11. Akses wazuh manager di web browser dengan
    ```
    https://<ALAMAT_IP_UBUNTU_SERVER_ANDA>
    ```
    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7f59f9af-7aae-4245-bed7-d336acc83810" />

12. Untuk login, coba dengan username dan password 'admin'. Jika gagal, jalankan perintah berikut:
    ```
     sudo tar -axf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt -O | grep -P "\'admin\'" -A 1
    ```
    Berikut adalah tampilan dashboard wazuh manager ketika pertama kali berhasil login
    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e6b0ad46-7ae1-4dc2-9f5f-7f9634268f7c" />

#  Instal Wazuh Agent, Deploy Website, Install WAF ModSecurity, dan Suricata IDS di Ubuntu Server (Server Target)
1. Cara installasi ubuntu server di vm untuk server target sama dengan instalasi ubuntu server untuk server SIEM. Namun, untuk server target ini spesifikasinya lebih ringan yaitu : Base Memory : 2GB dan Processors : 1 Core.
2. Klik Start
3. Pertama, installasi wazuh agent menggunakan perintah berikut
   ```
   curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
   echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

   sudo apt update
   sudo apt install wazuh-agent -y
   ```
4. Konfigurasi agent
   ```
   sudo nano /var/ossec/etc/ossec.conf
   ```
   edit bagian
   ```
   <client>
   <server>
    <address>IP_WAZUH_SERVER</address>
     </server>
   </client>
   ```
5. Kembali ke server SIEM, jalankan perintah 
   ```
   sudo /var/ossec/bin/manage_agents
   ```
   Tambahkan agent
   <img width="422" height="235" alt="image" src="https://github.com/user-attachments/assets/5ca1eaa7-640c-4dec-b94b-63f9c50ce1d1" />

   copy key
   <img width="473" height="302" alt="image" src="https://github.com/user-attachments/assets/41cd4495-942c-4b9f-828e-5bfaa1ea9571" />

 6. Kembali ke server target:
   ```
   sudo /var/ossec/bin/agent-auth -m IP_WAZUH_SERVER
   ```
7. Setup Web Server di Target, install apache
   ```
   sudo apt install apache2 -y
   ```
8. Verifikasi instalasi apache2
   ```
   sudo systemctl status apache2
   ```
9. Jika belum aktif, jalankan perintah
   ```
   sudo systemctl start apache2
   ```
10. Agar Apache langsung menyala setiap kali server di-reboot, jalankan:
    ```
    sudo systemctl enable apache2
    ```
11. Siapkan folder website
    ```
    sudo mkdir -p /var/www/website-kamu
    ```
12. Pindahkan file website hasil coding ke folder tadi menggunakan SCP, FileZilla, atau kalau cuma buat ngetes, bikin file baru:
    ```
    sudo nano /var/www/website-kamu/index.html
    ```
    Isi dengan kode HTML yang diinginkan
13. Atur hak akses (permission) supaya Apache bisa membaca file tanpa kendala keamanan. Kita berikan kepemilikan folder ke user www-data (usernya si Apache).
    ```
    sudo chown -R www-data:www-data /var/www/website-kamu
    sudo chmod -R 755 /var/www/website-kamu
    ```
14. Buat virtual host, ini bagian paling krusial. Apache perlu tahu kalau ada orang akses alamat tertentu, harus diarahkan ke folder mana. Bikin konfigurasi baru
    ```
    sudo nano /etc/apache2/sites-available/website-kamu.conf
    ```
    isi dengan kode berikut
       ```
       <VirtualHost *:80>
          ServerAdmin admin@website.com
          DocumentRoot /var/www/website-kamu
          ServerName alamat-ip-server-kamu
   
          ErrorLog ${APACHE_LOG_DIR}/error.log
          CustomLog ${APACHE_LOG_DIR}/access.log combined
       </VirtualHost>
      ```
15. Aktifkan config baru
    ```
      sudo a2ensite website-kamu.conf
    ```
16. Matikan config default (biar gak tabrakan)
    ```
      sudo a2dissite 000-default.conf
    ```
17. Cek apakah ada error penulisan
    ```
    sudo apache2ctl configtest
    ```
18. Akses website yang sudah dideploy tadi di web browser dengan
    ```
    http://<ALAMAT_IP_SERVER>
    ```
    <img width="958" height="756" alt="image" src="https://github.com/user-attachments/assets/24f5209f-31c7-480c-b92c-c65d20a94be3" />

19. Instalasi WAF (ModSecurity)
    ```
      sudo apt install libapache2-mod-security -y
    ```
20. Aktifkan
    ```
    a2enmod security2
    ```
21. Restart apache
    ```
    sudo systemctl restart apache2
    ```
Secara default, ModSecurity baru "terpasang" tapi belum "bekerja" memblokir. Dia cuma menonton saja (Detection Only). Yuk, kita suruh dia jadi satpam galak.
22. Copy file konfigurasi dasar
    ```
    sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
    ``` 
23. Edit filenya
    ```
    sudo nano /etc/modsecurity/modsecurity.conf
    ```
    Cari tulisan SecRuleEngine DetectionOnly, lalu ubah menjadi:SecRuleEngine On. Simpan (Ctrl+O, Enter) dan Keluar (Ctrl+X). ModSecurity butuh daftar "aturan" untuk tahu mana serangan dan mana yang bukan. Kita akan pakai OWASP Core Rule Set (CRS) yang sudah sangat lengkap.
24. Hapus aturan bawaan yang lama
    ```
    sudo rm -rf /usr/share/modsecurity-crs
    ```
25. Download aturan terbaru dari github
    ```
     sudo git clone https://github.com/coreruleset/coreruleset /usr/share/modsecurity-crs
    ```
26. Siapkan file configurasinya
    ```
    sudo mv /usr/share/modsecurity-crs/crs-setup.conf.example /usr/share/modsecurity-crs/crs-setup.conf
    ```
27. Beritahu apache untuk memakai aturan tersebut, dengan edit file  /etc/apache2/mods-enabled/security2.conf
    ```
    sudo nano /etc/apache2/mods-enabled/security2.conf
    ```
    Pastikan di dalamnya ada baris ini (arahkan ke folder yang baru kita download):
    ```
    IncludeOptional /etc/modsecurity/*.conf
    Include /usr/share/modsecurity-crs/crs-setup.conf
    Include /usr/share/modsecurity-crs/rules/*.conf
    ```
28. Restart Apache
    ```
    sudo systemctl restart apache2
    ```
    Di dalam case saya, ada eror dikarenakan ubuntu yang saya gunakan lebih jadul dari pada owasp crs yang baru di download. Untuk mengakalinya, gunakan perintah berikut
    ```
    //Masuk ke folder rules
    cd /usr/share/modsecurity-crs/rules/
    //Ubah nama file yang bermasalah dengan ganti akhiran .conf jadi .bak supaya tidak dibaca oleh Apache.
    sudo mv REQUEST-922-MULTIPART-ATTACK.conf REQUEST-922-MULTIPART-ATTACK.conf.bak
    ```
    Tes apache
    ```
    sudo apache2ctl configtest
    ```
    Kalau muncul "Syntax OK" berarti sudah berhasil, tinggal jalankan kembali perintah untuk restart apachenya. 
Sekarang, kita ingin si satpam (ModSecurity) melapor ke pengawas (Wazuh) kalau ada serangan.
1. Edit konfigurasi wazuh agent
   ```
   sudo nano /var/ossec/etc/ossec.conf
   ```
2. Tambahkan blok berikut di bagian bawah (sebelum penutup </ossec_config>):
   ```
   <localfile>
      <log_format>apache</log_format>
      <location>/var/log/apache2/error.log</location>
    </localfile>

    <localfile>
      <log_format>json</log_format>
      <location>/var/log/apache2/modsec_audit.log</location>
    </localfile>
   ```
3. Restart Wazuh Agent
   ```
   sudo systemctl restart wazuh-agent
   ```





    






















