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
a. Server SIEM : Server untuk SIEM manajer 
b. Server Target : Server yang akan digunakan untuk Deploy Website, Install WAF ModSecurity, Wazuh Agent, dan Suricata IDS
c. Server Attacker : Server yang digunakan untuk menyerang 

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
12. Untuk login, coba dengan username dan password 'admin'. Jika gagal, jalankan perintah berikut:
    ```
     sudo tar -axf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt -O | grep -P "\'admin\'" -A 1
    ```
# Deploy Website, Install WAF ModSecurity, Wazuh Agent, dan Suricata IDS di Ubuntu Server (Server Target)
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
7. Setup Web Server di Target























