**Bug Bounty Recon Methodology 2025**  
**Part 1**  

**Author:** SilverX  

**Topics:**  
- Passive Subdomain Enumeration  
- Active Subdomain Enumeration  
- URL Discovery  
- Network Analysis (Reverse DNS, ASN Lookup, ASN Enumeration)  
- Cloud Enumeration  

**Note:** The more subdomains we have for a target site, the more potential vulnerabilities we can discover.  

---

### **Passive Subdomain Enumeration**  
During passive subdomain enumeration, no direct queries are sent to the target. Instead, information is gathered from open sources.

**Command 1:**  
```bash
subfinder -d target.com -silent -all -recursive -o subfinder_result.txt  
```  
**Explanation:** The `subfinder` tool is used to passively collect subdomains of the target. The `-d` parameter specifies the target domain, `-silent` shows only the results, `-all` checks all data sources, `-recursive` performs a recursive search, and the results are saved to `subfinder_result.txt`.  

**Command 2:**  
```bash
amass enum -passive -d target.com -o amasspassive_result.txt  
```  
**Explanation:** The `amass` tool is used to passively collect subdomains. The `-passive` parameter ensures no direct queries are sent, `-d` specifies the target domain, and the results are saved to `amasspassive_result.txt`.  

**Command 3:**  
```bash
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | anew crtsh_result.txt  
```  
**Explanation:** Subdomains are collected from SSL certificates via `crt.sh`. The `curl` command sends the request, `jq` parses the JSON data, `sed` removes unnecessary characters, and the results are saved to `crtsh_result.txt`.  

**Command 4:**  
```bash
github-subdomains -d target.com -t github_api_token -o github_subs.txt  
```  
**Explanation:** The `github-subdomains` tool collects subdomains from open-source code on GitHub. The `-d` parameter specifies the target domain, `-t` specifies the GitHub API token, and the results are saved to `github_subs.txt`.  

**Command 5:**  
```bash
cat *.txt | sort -u > all_subs.txt
```  
**Explanation:** All subdomains are combined into a single file. The `sort -u` command ensures duplicate subdomains are removed.  

**Command 7:**  
```bash
cat all_subs.txt | httpx -silent -title -o live_subdomains.txt
```  
**Explanation:** Live subdomains are filtered and saved to a separate file.  

---

### **Active Subdomain Enumeration**  
During active enumeration, direct queries are sent to the target. This method is more effective but may attract the target's attention.  

**Command 1:**  
```bash
cat /usr/share/SecLists/Discovery/DNS/subdomain-top1million-5000.txt | sed 's/$/.target.com/' > wordlist.txt  
```  
**Explanation:** A wordlist for subdomains is prepared using the `SecLists` library. The `sed` command appends the target domain to each line, and the results are saved to `wordlist.txt`.  

**Command 2:**  
```bash
massdns -r /home/silverx/Downloads/massdns/lists/resolvers.txt -t A -o S -w massdns_result.txt wordlist.txt  
```  
**Explanation:** The `massdns` tool sends DNS queries. The `-r` parameter specifies DNS resolvers, `-t A` requests A records, `-o S` shows results in a simple format, and the results are saved to `massdns_result.txt`.  

**Command 3:**  
```bash
shuffledns -d target.com -l all_subs.txt -r /home/silverx/Downloads/shuffledns/tests/resolvers.txt -o active_subs.txt -mode resolve  
```  
**Explanation:** The `shuffledns` tool checks subdomains. The `-d` parameter specifies the target domain, `-l` specifies the subdomain list, `-r` specifies DNS resolvers, and the results are saved to `active_subs.txt`.  

**Command 4:**  
```bash
dnsx -l active_subs.txt -resp -o resolved_subs.txt  
```  
**Explanation:** The `dnsx` tool checks subdomains and filters responding ones. The `-l` parameter specifies the subdomain list, `-resp` filters responding subdomains, and the results are saved to `resolved_subs.txt`.  

**Command 5:**  
```bash
ffuf -u https://FUZZ.target.com -w subdomains.txt -t 50 -mc 200,403 -o ffuf_result.txt  
```  
**Explanation:** The `ffuf` tool performs fuzzing on subdomains. The `-u` parameter specifies the URL format, `-w` specifies the wordlist, `-t` specifies the number of threads, `-mc` filters response codes, and the results are saved to `ffuf_result.txt`.  

---

### **URL Discovery**  
During the URL discovery phase, the target's historical and current endpoints are identified.  

**Command 1:**  
```bash
gau target.com | anew target_gauresults.txt  
```  
**Explanation:** The `gau` tool collects historical URLs used by the target. The results are saved to `target_gauresults.txt`.  

**Command 2:**  
```bash
waybackurls target.com | anew target_waybackurlsresults.txt  
```  
**Explanation:** The `waybackurls` tool collects URLs from the Wayback Machine. The results are saved to `target_waybackurlsresults.txt`.  

**Command 3:**  
```bash
katana -u target.com -silent -jc -o target_katanaresult.txt  
```  
**Explanation:** The `katana` tool discovers the target's current endpoints. The `-u` parameter specifies the target URL, `-silent` shows only results, `-jc` checks JavaScript files, and the results are saved to `target_katanaresult.txt`.  

**Command 4:**  
```bash
echo "https://target.com" | hakrawler -depth 2 -plain -js -out target_hakrawlerresults.txt  
```  
**Explanation:** The `hakrawler` tool explores the target's URLs. The `-depth` parameter specifies the depth, `-plain` shows results in a simple format, `-js` checks JavaScript files, and the results are saved to `target_hakrawlerresults.txt`.  

---

### **Network Analysis**  
During the network analysis phase, the target's network structure, DNS records, and ASN information are analyzed.  

**Command 1 (Reverse DNS):**  
```bash
dnsx -ptr -l resolved_subs.txt -resp-only -o reverse_dns.txt  
```  
**Explanation:** The `dnsx` tool performs PTR queries to find domains corresponding to IP addresses. The results are saved to `reverse_dns.txt`.  

**Command 2 (ASN Lookup):**  
```bash
curl https://ipinfo.io/ip_address  
```  
**Explanation:** The `curl` command is used to find the ASN by querying the target's origin IP address. Alternatively, you can perform a WHOIS lookup on the origin IP.  

**Command 3 (ASN Enumeration):**  
```bash
amass intel -asn <ASN_NUMBER> -o asn_results.txt  
```  
**Explanation:** The `amass` tool collects network information based on the ASN number. The results are saved to `asn_results.txt`.  

---

### **Cloud Enumeration**  
During cloud enumeration, the target's cloud-based services and configurations are analyzed.  

**Command 1:**  
```bash
cloud_enum -k target.com -f cenum_result.txt  
```  
**Explanation:** The `cloud_enum` tool discovers the target's cloud resources. The `-k` parameter specifies the keyword, and the `-f` parameter specifies the output file.  

---

### **Conclusion**  
This is Part 1. If supported, Part 2 will be written. Thank you.  

---  
**Author:** SilverX
