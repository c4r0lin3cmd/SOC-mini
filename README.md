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

# Installasi SIEM Wazuh di Ubuntu server 22.04.5 (VM SIEM)
1. Buka VM > New > Isi Vm Name, VM Folder sesuaikan dengan penyimpanan yang diinginkan > Select ISO Image dengan mengklik tanda panah kebawah dan pilih iso ubuntu server yang sudah di download.
   <img width="946" height="761" alt="image" src="https://github.com/user-attachments/assets/1d0462a2-bbc1-4885-99ad-1a979516014a" />

3. Atur username, password, serta hostname yang diinginkan
4. Atur 'Specify virtual hardware' dengan spesifikasi sebagai berikut
   Base Memory : 4GB
   Processors  : 2 core
   > Klik Finish
5. Klik Setting > Network buka di tab Expert > Pastikan pada Adapter 1 attachednya 'Bridged Adapter'
   <img width="1161" height="757" alt="image" src="https://github.com/user-attachments/assets/3af46617-e9cf-4cde-9a4e-1344c91cfd86" />
6. Update dan Upgrade package
   ```
   sudo apt update && sudo apt upgrade -y
   ```
7. install wazuh langsung dari sumber resimnya
   ```
   curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
   ```
8. Verifikasi installasi wazuh dengan perintah
   ```
   sudo systemctl status wazuh-manager
   ```
   Jika status active (running) maka wazuh sudah siap digunakan.
9. Pada terminal ubuntu server, jalankan perintah
    ```
    ip a
    ```
    Lihat pada enp0 > inet itu adalah alamat ip ubuntu servernya
10. Akses wazuh manager di web browser dengan
    ```
    https://<ALAMAT_IP_UBUNTU_SERVER_ANDA>
    ```
