# Digital-gatekeeper
Bu repoda Bilgi ve ağ güvenliği dersi projesi "Digital gatekeeper" anlatılmıştır.

Arkadaşlar merhaba Digital gatekeeper projesinin 13. hafta konusu GÜVENLİ YÖNETİM VE LOG ERİŞİMİ (VPN):

Aşama 1: Kali'de WireGuard Sunucu Kurulumu:

1.1: Paket kurulumu için terminalde bu kodu çalıştırıyoruz: "sudo apt update && sudo apt install wireguard -y"

1.2: Anahtar üretimi için bu kodları  çalıştırıyoruz biri sunucu(kali) diğeri istemci(windows) için:" cd /etc/wireguard , wg genkey | sudo tee server_private.key | wg
pubkey | sudo tee server_public.key , sudo chmod 600 server_private.key , wg genkey | sudo tee client_private.key | wg pubkey | sudo tee client_public.key, sudo chmod 600 client_private.key"

Ardından "sudo cat server_private.key 
sudo cat server_public.key    
sudo cat client_private.key   
sudo cat client_public.key"
komutları ile keyleri görüntüleyip not ediyoruz.

1.3: "sudo nano /etc/wireguard/wg0.conf" komutu ile sunucu yapılandırması için bir config dosyası oluşturuyoruz.
Ardından config dosyasının içine:
"[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>

[Peer]
PublicKey = <CLIENT_PUBLIC_KEY>
AllowedIPs = 10.0.0.2/32"

Komutlarını ekleyip  servisin içeriğini oluşturuyoruz. Burada dikkat edilmesi gereken konu "interface" kısmını cihazın kendi bilgileri ile "peer" kısmını iletişime geçilecek cihazın bilgileri ile dolduruyoruz.

1.4: servisi başlatma ve sistem açılışında otomatik başlatılmasını sağlamak için gerekli komutlar: "sudo wg-quick up wg0(sistemi başlatma), sudo wg show(servisin durumunu gösterme), sudo systemctl enable wg-quick@wg0(servisin sistem ile birlikte otomatik açılmasını sağlama).

1.5: Firewallda port açıyoruz:"iptables -A INPUT -p udp --dport 51820 -j ACCEPT" 51820 burada dinleyeceğimiz port ve bu komutla porttan gelen paketlerin kabul edilmesini sağlıyoruz ardından "apt install iptables-persistent -y,
netfilter-persistent save"
komutları ile kaydederek portumuzun kalıcı olmasını sağlıyoruz.

Aşama 2. Windows wireguard kurulumu:

2.1: "https://www.wireguard.com/install/" linkinden windows bilgisayarımız için wireguard indiriyoruz.

2.2: Wireguard uygulamasına boş bir tünel ekliyoruz ardından tüneli düzenleyip içine :

[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>  "kali üzerinden oluşturduğumuz istemci private keyi"
Address = 10.0.0.2/32 "istemci ip adresi"

[Peer]
PublicKey = <SERVER_PUBLIC_KEY> "kali üzerinden oluşturduğumuz sunucu public keyi"
Endpoint = <KALI_GERCEK_IP>:51820 "kalinin iç ağ ipsi"
AllowedIPs = 10.0.0.0/24 "sunucu ip adresi"
PersistentKeepalive = 25 "bağlantının kompadan sağlıklı bir şekilde devam etmesi için gerekli kod"

komutlarını ekliyoruz.

Ardından activate düğmesine basarak bağlantıyı aktif hale getiriyoruz
<img width="657" height="516" alt="image" src="https://github.com/user-attachments/assets/e0ca81ed-11d2-4fb9-92b0-7e728e177d4a" />

2.3: Bağlantı testi: windowsta cmd üzerinden sunucu ipsine ping atarak bağlantımızı test ediyoruz:
<img width="459" height="170" alt="image" src="https://github.com/user-attachments/assets/60f13552-1aa2-40b8-bdf5-4abaa7dfce8f" />
Pingimize yanıt geldiğini görüyoruz yani bağlantı başarılı ve aktif.
Ayrıca kali üzerinden de "sudo wg show" komutu ile handshakein gerçekleştiğini görebiliriz:
<img width="860" height="301" alt="image" src="https://github.com/user-attachments/assets/fcc11a96-9f9b-44d5-9ce7-f532608db1d9" />



Aşama 3: VPN Üzerinden SSH ile Log Erişimi:

Bağlantıyı aktif ettikten sonra windowsta powershell üzerinden kaliye bağlanıyoruz :

<img width="659" height="232" alt="image" src="https://github.com/user-attachments/assets/c2a51f08-c1a6-4c76-aadc-7949f562155a" />

bağlandıktan sonra dosyalar içerisinde gezinip istediğimiz klasörleri ve dosyaları gezip logları okuyabiliyoruz.

<img width="328" height="82" alt="image" src="https://github.com/user-attachments/assets/54959f45-7531-4fc0-9c23-80926cd7d80f" />

burada log dosyası henüz oluşmadığı için boş gözüküyor fakat erişebildiğimiz için okuyabildiğimizi kanıtlıyoruz.

Aşama 4: Snort'un wg0 Arayüzünü İzlemesi:

4.1: Snort'un wg0 arayüzünü dinlemesini başlatıyoruz:
"sudo snort -R /etc/snort/rules/local.rules -i wg0 -A alert_fast -k none" 

ardından dinleme vpn üzerinde oluşan trafiği tutar ve dinlemeyi durdurduğumuzda bize listeler:
<img width="911" height="763" alt="image" src="https://github.com/user-attachments/assets/28c5a37d-54c6-44b2-a17b-04ce5dfd0610" />
<img width="907" height="819" alt="image" src="https://github.com/user-attachments/assets/2d7374df-b195-4822-a7f6-398d7248e9ca" />









.
