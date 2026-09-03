# Analise de access.log -- Filipe Rezende Silva (@filipesilva07)

**Linhas analisadas:** 516866

## 1. Volume e falha

```bash
awk '{
    total++;
    if ($9 ~ /^4[0-9][0-9]$/) e4++;
    if ($9 ~ /^5[0-9][0-9]$/) e5++;
} END {
    printf "Total: %d\n4xx: %d (%.2f%%)\n5xx: %d (%.2f%%)\n", total, e4, e4/total*100, e5, e5/total*100
}' dados/access.log
Total: 516866
4xx: 6162 (1.19%)
5xx: 11749 (2.27%)

awk '{print $1}' dados/access.log | sort | uniq -c | sort -rn | head -10

88400 203.0.113.47
 1788 192.0.2.245
 1772 192.0.2.171
 1771 192.0.2.81
 1771 192.0.2.225
 1771 192.0.2.16
 1767 192.0.2.138
 1762 192.0.2.222
 1757 192.0.2.45
 1753 192.0.2.166

awk '$1 == "203.0.113.47" {print $7}' dados/access.log | sort | uniq -c | sort -rn | head -10

22224 /api/busca?q=mochila
22161 /api/busca?q=tenis
22090 /api/busca?q=camiseta
21925 /api/busca?q=fone

awk '$1 == "203.0.113.47" {print $9}' dados/access.log | sort | uniq -c | sort -rn

81500 200
 6900 503

awk '$1 == "203.0.113.47" {total++; if ($9 == 503) erros++} END {printf "Total: %d\n503: %d (%.2f%%)\n", total, erros, erros/total*100}' dados/access.log

Total: 88400
503: 6900 (7.81%)

awk '$1 == "203.0.113.47" {print $12}' dados/access.log | sort | uniq -c | sort -rn

88400 "curl/8.5.0"

awk '$9 == 500 {print $7}' dados/access.log | sort | uniq -c | sort -rn | head -10

3620 /api/relatorio/gerar
 228 /
 167 /api/produtos
 160 /produtos
 150 /produtos/detalhe
 117 /static/app.css
 105 /static/app.js
  92 /api/carrinho
  63 /api/busca
  44 /favicon.ico

awk '$9 == 500 {total++} $9 == 500 && $7 == "/api/relatorio/gerar" {endpoint++} END {printf "Total 500: %d\n/api/relatorio/gerar: %d (%.2f%%)\n", total, endpoint, endpoint/total*100}' dados/access.log

Total 500: 4849
/api/relatorio/gerar: 3620 (74.65%)


awk '$7 == "/api/relatorio/gerar" {total++} END {print total}' dados/access.log

10400

awk '{print $4}' dados/access.log | cut -d: -f2 | sort | uniq -c | sort -rn

68535 23
43979 22
32529 15
32526 11
31895 12
31225 14
30886 16
30869 10
29952 13
28575 17
27621 09
25996 18
22807 19
19519 08
18860 20
15577 21
 9759 07
 3904 06
 3262 00
 1967 01
 1844 05
 1810 02
 1498 04
 1471 03

awk '$7 ~ /^\/(admin|\.env|\.git|wp-login|phpmyadmin)/ {print $7}' dados/access.log | sort | uniq -c | sort -rn

382 /admin/login
368 /wp-login.php
356 /.git/config
343 /.env
318 /phpmyadmin/index.php
313 /admin

awk '$7 ~ /^\/(admin|\.env|\.git|wp-login|phpmyadmin)/ {print $1}' dados/access.log | sort -u | wc -l

2


