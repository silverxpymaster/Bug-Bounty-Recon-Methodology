**Bug Bounty Recon Methodology 2025**  
**Part 1**  

**Author:** SilverX  

**Mövzular:**  
- Passive Subdomain Enumeration  
- Active Subdomain Enumeration  
- URL Discovery  
- Network Analysis (Reverse DNS, ASN tapma, ASN Enumeration)  
- Cloud Enumeration  

**Qeyd:** Bir hədəf saytın nə qədər çox subdomaini əlimizdə olsa bir o qədər zəiflik axtara biləcəyimiz yer sayı artar.  

---

### **Passive Subdomain Enumeration**  
Passive subdomain enumeration zamanı heç bir birbaşa sorğu hədəfə göndərilmir.Bunun əvəzinə açıq mənbələrdən məlumat toplanır

**Komanda 1:**  
```bash
subfinder -d target.com -silent -all -recursive -o subfinder_result.txt  
```  
**İzahat:** `subfinder` aləti vasitəsilə hədəfin subdomainləri passiv şəkildə toplanır.`-d` parametri ilə hədəf domain `-silent` parametri ilə yalnız nəticələr göstərilir `-all` parametri ilə bütün məlumat mənbələri yoxlanılır `-recursive` parametri ilə rekursiv axtarış edilir və nəticələr `subfinder_result.txt` faylına yazılır.  

**Komanda 2:**  
```bash
amass enum -passive -d target.com -o amasspassive_result.txt  
```  
**İzahat:** `amass` aləti ilə passiv şəkildə subdomainlər toplanır. `-passive` parametri ilə heç bir birbaşa sorğu göndərilmir, `-d` parametri ilə hədəf domain qeyd edilir və nəticələr `amasspassive_result.txt` faylına yazılır.  

**Komanda 3:**  
```bash
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | anew crtsh_result.txt  
```  
**İzahat:** `crt.sh` saytından SSL sertifikatları vasitəsilə subdomainlər toplanır.`curl` ilə sorğu göndərilir `jq` ilə JSON məlumatları parse edilir `sed` ilə lazımsız simvollar təmizlənir və nəticələr `crtsh_result.txt` faylına yazılır.  

**Komanda 4:**  
```bash
github-subdomains -d target.com -t github_api_token -o github_subs.txt  
```  
**İzahat:** `github-subdomains` aləti ilə GitHub-da olan açıq mənbə kodlarından subdomainlər toplanır.`-d` parametri ilə hədəf domain `-t` parametri ilə GitHub API tokeni qeyd edilir və nəticələr `github_subs.txt` faylına yazılır.  

**Komanda 5:**  
```bash
cat *.txt | sort -u > all_subs.txt
```  
**İzahat:** `butun subdomainler eyni faylda toplanir sort -u  emri ile eyni subdomainlerin yazilmamasini temin edirik

**Komanda 7:**  
```bash
cat all_subs.txt | httpx -silent -title -o live_subdomains.txt
```  
**İzahat:** `Canli subdomainleri ayirib basqa txt faylina yaziriq
---



### **Active Subdomain Enumeration**  
Active enumeration zamanı hədəfə birbaşa sorğular göndərilir.Bu üsul daha effektiv olsa da hədəfin diqqətini cəlb edə bilər

**Komanda 1:**  
```bash
cat /usr/share/SecLists/Discovery/DNS/subdomain-top1million-5000.txt | sed 's/$/.target.com/' > wordlist.txt  
```  
**İzahat:** `SecLists` kitabxanasından subdomainlər üçün wordlist hazırlanır.`sed` ilə hər bir sətirin sonuna hədəf domain əlavə edilir və nəticələr `wordlist.txt` faylına yazılır.  

**Komanda 2:**  
```bash
massdns -r /home/silverx/Downloads/massdns/lists/resolvers.txt -t A -o S -w massdns_result.txt wordlist.txt  
```  
**İzahat:** `massdns` aləti ilə DNS sorğuları göndərilir. `-r` parametri ilə DNS resolverləri qeyd edilir, `-t A` ilə A qeydləri soruşulur, `-o S` ilə sadə formatda nəticələr göstərilir və nəticələr `massdns_result.txt` faylına yazılır.  

**Komanda 3:**  
```bash
shuffledns -d target.com -l all_subs.txt -r /home/silverx/Downloads/shuffledns/tests/resolvers.txt -o active_subs.txt -mode resolve  
```  
**İzahat:** `shuffledns` aləti ilə subdomainlər yoxlanılır. `-d` parametri ilə hədəf domain, `-l` parametri ilə subdomain siyahısı, `-r` parametri ilə DNS resolverləri qeyd edilir və nəticələr `active_subs.txt` faylına yazılır.  

**Komanda 4:**  
```bash
dnsx -l active_subs.txt -resp -o resolved_subs.txt  
```  
**İzahat:** `dnsx` aləti ilə subdomainlər yoxlanılır və cavab verənlər ayrılır. `-l` parametri ilə subdomain siyahısı, `-resp` parametri ilə cavab verənlər seçilir və nəticələr `resolved_subs.txt` faylına yazılır.  

**Komanda 5:**  
```bash
ffuf -u https://FUZZ.target.com -w subdomains.txt -t 50 -mc 200,403 -o ffuf_result.txt  
```  
**İzahat:** `ffuf` aləti ilə subdomainlər üzrə fuzzing edilir. `-u` parametri ilə URL formatı, `-w` parametri ilə wordlist qeyd edilir, `-t` parametri ilə thread sayı, `-mc` parametri ilə cavab kodları seçilir və nəticələr `ffuf_result.txt` faylına yazılır.  

---

### **URL Discovery**  
URL discovery mərhələsində hədəfin keçmişdə istifadə etdiyi URL-lər və mövcud endpointlər aşkar edilir

**Komanda 1:**  
```bash
gau target.com | anew target_gauresults.txt  
```  
**İzahat:** `gau` aləti ilə hədəfin keçmişdə istifadə etdiyi URL-lər toplanır. Nəticələr `target_gauresults.txt` faylına yazılır.  

**Komanda 2:**  
```bash
waybackurls target.com | anew target_waybackurlsresults.txt  
```  
**İzahat:** `waybackurls` aləti ilə Wayback Machine-dən URL-lər toplanır. Nəticələr `target_waybackurlsresults.txt` faylına yazılır.  

**Komanda 3:**  
```bash
katana -u target.com -silent -jc -o target_katanaresult.txt  
```  
**İzahat:** `katana` aləti ilə hədəfin mövcud endpointləri aşkar edilir. `-u` parametri ilə hədəf URL, `-silent` parametri ilə yalnız nəticələr göstərilir, `-jc` parametri ilə JavaScript faylları yoxlanılır və nəticələr `target_katanaresult.txt` faylına yazılır.  

**Komanda 4:**  
```bash
echo "https://target.com" | hakrawler -depth 2 -plain -js -out target_hakrawlerresults.txt  
```  
**İzahat:** `hakrawler` aləti ilə hədəfin URL-ləri kəşf edilir. `-depth` parametri ilə dərinlik, `-plain` parametri ilə sadə format, `-js` parametri ilə JavaScript faylları yoxlanılır və nəticələr `target_hakrawlerresults.txt` faylına yazılır.  

---

### **Network Analysis**  
Network analysis mərhələsində hədəfin şəbəkə quruluşu, DNS qeydləri və ASN məlumatları təhlil edilir.  

**Komanda 1 (Reverse DNS):**  
```bash
dnsx -ptr -l resolved_subs.txt -resp-only -o reverse_dns.txt  
```  
**İzahat:** `dnsx` aləti ilə PTR sorğuları vasitəsilə IP ünvanlarına uyğun domainlər tapılır. Nəticələr `reverse_dns.txt` faylına yazılır.  

**Komanda 2 (ASN Tapma):**  
```bash
curl https://ipinfo.io/ip_address  
```  
**İzahat:** `curl` komutu ile ip_address yazdigm yere hedef saytin origin ip-sini yazib ASN oyrenirsiz veyaxud origin ip-ye whois ata bilersiniz 

**Komanda 3 (ASN Enumeration):**  
```bash
amass intel -asn <ASN_NUMBER> -o asn_results.txt  
```  
**İzahat:** `amass` aləti ilə ASN nömrəsinə uyğun şəbəkə məlumatları toplanır. Nəticələr `asn_results.txt` faylına yazılır.  

---

### **Cloud Enumeration**  
Cloud enumeration zamanı hədəfin bulud əsaslı xidmətləri və konfiqurasiyaları təhlil edilir.  

**Komanda 1:**  
```bash
cloud_enum -k target.com -f cenum_result.txt  
```  
**İzahat:** `cloud_enum` aləti ilə hədəfin bulud resursları aşkar edilir. `-k` parametri ilə açar söz, `-f` parametri ilə nəticələr faylı qeyd edilir.  

---

### **Yekun**  
Bu part-1 oldu , Destek olunsa part 2de yazilacaq.Tesekkurler

---  
**Author:** SilverX  
