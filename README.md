**Bug Bounty Recon Methodology 2025**  
**Part 1**  

**Author:** SilverX  

**Mövzular:**  
- Passive Subdomain Enumeration  
- Active Subdomain Enumeration  
- URL Discovery  
- Network Analysis (Reverse DNS, ASN tapma, ASN Enumeration)  
- Cloud Enumeration  

**Qeyd:** Bir hədəf saytın nə qədər çox subdomaini əlimizdə olsa, bir o qədər zəiflik axtara biləcəyimiz yer sayı artar.  

---

### **Passive Subdomain Enumeration**  
Passive subdomain enumeration zamanı heç bir birbaşa sorğu hədəfə göndərilmir. Bunun əvəzinə, ictimai məlumat bazaları, sertifikat şəbəkələri və digər açıq mənbələrdən məlumat toplanır.  

**Nümunə komandalar:**  
```bash
subfinder -d rapfame.app -silent -all -recursive -o subfinder_result.txt  

amass enum -passive -d rapfame.app -o amasspassive_result.txt  

curl -s "https://crt.sh/?q=%25.rapfame.app&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | anew crtsh_result.txt  

github-subdomains -d target.com -t github_api_token -o github_subs.txt  
```  

**Əlavə Məlumat:**  
Passive enumeration zamanı ən çox istifadə olunan vasitələr arasında `subfinder`, `amass`, `crt.sh` və `github-subdomains` kimi alətlər var. Bu alətlər vasitəsilə hədəfin subdomainləri əsasən açıq mənbələrdən toplanır. Bu üsul hədəfə heç bir təsir göstərmədiyi üçün təhlükəsiz hesab olunur.  

---

### **Active Subdomain Enumeration**  
Active enumeration zamanı hədəfə birbaşa sorğular göndərilir. Bu üsul daha effektiv olsa da, hədəfin diqqətini cəlb edə bilər.  

**Nümunə komandalar:**  
```bash
cat /usr/share/SecLists/Discovery/DNS/subdomain-top1million-5000.txt | sed 's/$/.rapfame.app/' > wordlist.txt  

massdns -r /home/silverx/Downloads/massdns/lists/resolvers.txt -t A -o S -w massdns_result.txt wordlist.txt  

shuffledns -d rapfame.app -l all_subs.txt -r /home/silverx/Downloads/shuffledns/tests/resolvers.txt -o active_subs.txt -mode resolve  

dnsx -l active_subs.txt -resp -o resolved_subs.txt  

ffuf -u https://FUZZ.rapfame.app -w subdomains.txt -t 50 -mc 200,403 -o ffuf_result.txt  
```  

**Əlavə Məlumat:**  
Active enumeration zamanı `massdns`, `shuffledns`, `dnsx` və `ffuf` kimi alətlər geniş istifadə olunur. Bu alətlər vasitəsilə hədəfin DNS qeydləri tərəfli şəkildə yoxlanılır və subdomainlər aşkar edilir. Bu üsul daha dəqiq nəticələr versə də, hədəfin müdafiə mexanizmlərini aktivləşdirə bilər.  

---

### **URL Discovery**  
URL discovery mərhələsində hədəfin keçmişdə istifadə etdiyi URL-lər və mövcud endpointlər aşkar edilir.  

**Nümunə komandalar:**  
```bash
gau target.repfame.app | anew target_gauresults.txt  

waybackurls target.rapfame.app | anew target_waybackurlsresults.txt  

katana -u target.rapfame.app -silent -jc -o target_katanaresult.txt  

echo "https://target.rapfame.app" | hakrawler -depth 2 -plain -js -out target_hakrawlerresults.txt  
```  

**Əlavə Məlumat:**  
URL discovery zamanı `gau`, `waybackurls`, `katana` və `hakrawler` kimi alətlər geniş istifadə olunur. Bu alətlər vasitəsilə hədəfin keçmişdə istifadə etdiyi URL-lər, JavaScript faylları və digər endpointlər aşkar edilir. Bu məlumatlar zəifliklərin tapılması üçün əsas mənbə ola bilər.  

---

### **Network Analysis**  
Network analysis mərhələsində hədəfin şəbəkə quruluşu, DNS qeydləri və ASN məlumatları təhlil edilir.  

**Reverse DNS:**  
```bash
dnsx -ptr -l resolved_subs.txt -resp-only -o reverse_dns.txt  
```  

**ASN Tapma:**  
ASN tapmazdan əvvəl hədəfin origin IP ünvanını öyrənmək lazımdır. Bunun üçün aşağıdakı komandadan istifadə edə bilərsiniz:  
```bash
curl https://ipinfo.io/ip_address  
```  

**ASN Enumeration:**  
```bash
amass intel -asn <ASN_NUMBER> -o asn_results.txt  
```  

**Əlavə Məlumat:**  
Network analysis zamanı hədəfin şəbəkə infrastrukturu dərin şəkildə təhlil edilir. Reverse DNS sorğuları vasitəsilə hədəfin IP ünvanlarına uyğun domainlər tapılır. ASN tapma və enumeration isə hədəfin şəbəkə provayderi və digər əlaqəli məlumatlar haqqında məlumat verir.  

---

### **Cloud Enumeration**  
Cloud enumeration zamanı hədəfin bulud əsaslı xidmətləri və konfiqurasiyaları təhlil edilir.  

**Nümunə komanda:**  
```bash
cloud_enum -k rapfame.app -f cenum_result.txt  
```  

**Əlavə Məlumat:**  
Bulud əsaslı xidmətlər müasir dövrdə geniş yayıldığı üçün bu mərhələ xüsusi əhəmiyyət daşıyır. `cloud_enum` kimi alətlər vasitəsilə hədəfin AWS, Azure, Google Cloud kimi bulud provayderlərində olan resursları aşkar edilir. Bu resurslar zəifliklərin tapılması üçün potensial mənbə ola bilər.  

---

### **Yekun**  
Bug Bounty üçün reconnaissance mərhələsi hər zaman ən vacib mərhələlərdən biridir. Bu mərhələdə toplanan məlumatlar zəifliklərin tapılması üçün əsas rol oynayır. Passive və active üsulların düzgün kombinasiyası ilə hədəf haqqında maksimum məlumat toplamaq mümkündür. Həmçinin, network analysis və cloud enumeration kimi mərhələlər vasitəsilə hədəfin şəbəkə və bulud infrastrukturu dərin şəkildə təhlil edilə bilər.  

**Əlavə Tövsiyələr:**  
- Hər bir mərhələdə toplanan məlumatları düzgün şəkildə sənədləşdirin.  
- Fərqli alətlərin kombinasiyası ilə daha dəqiq nəticələr əldə edin.  
- Hədəfin müdafiə mexanizmlərini aktivləşdirməmək üçün sorğuları diqqətlə planlaşdırın.  

Bu metodologiya ilə hədəf haqqında maksimum məlumat toplayaraq, zəifliklərin aşkar edilməsi prosesini asanlaşdıra bilərsiniz. Təşəkkürlər!  

---  
**Yazar:** SilverX  
**Tarix:** 2025
