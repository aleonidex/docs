
| Ресурс        | Цена за 1 месяц,<br>вкл. НДС                             | Цена с CVoS на 6 месяцев,<br>вкл. НДС                                                 | Цена с CVoS на 1 год,<br>вкл. НДС                                                     |
|---------------|---------------------------------------------------------:|--------------------------------------------------------------------------------------:|--------------------------------------------------------------------------------------:|
| **Intel Broadwell**                                                                                                                                                                                                                                      |
| 5% vCPU       | {{ sku|RUB|mdb.cluster.mysql.v1.cpu.c5|month|string }}   | −                                                                                     | −                                                                                     |
| 20% vCPU      | {{ sku|RUB|mdb.cluster.mysql.v1.cpu.c20|month|string }}  | −                                                                                     | −                                                                                     |
| 50% vCPU      | {{ sku|RUB|mdb.cluster.mysql.v1.cpu.c50|month|string }}  | −                                                                                     | −                                                                                     |
| 100% vCPU     | {{ sku|RUB|mdb.cluster.mysql.v1.cpu.c100|month|string }} | −                                                                                     | −                                                                                     |
| RAM (за 1 ГБ) | {{ sku|RUB|mdb.cluster.mysql.v1.ram|month|string }}      | −                                                                                     | −                                                                                     |
| **Intel Cascade Lake**                                                                                                                                                                                                                                   |
| 5% vCPU       | {{ sku|RUB|mdb.cluster.mysql.v2.cpu.c5|month|string }}   | −                                                                                     | −                                                                                     |
| 20% vCPU      | {{ sku|RUB|mdb.cluster.mysql.v2.cpu.c20|month|string }}  | −                                                                                     | −                                                                                     |
| 50% vCPU      | {{ sku|RUB|mdb.cluster.mysql.v2.cpu.c50|month|string }}  | −                                                                                     | −                                                                                     |
| 100% vCPU     | {{ sku|RUB|mdb.cluster.mysql.v2.cpu.c100|month|string }} | {{ sku|RUB|v1.commitment.selfcheckout.m6.mdb.mysql.cpu.c100.v2|month|string }} (-15%) | {{ sku|RUB|v1.commitment.selfcheckout.y1.mdb.mysql.cpu.c100.v2|month|string }} (-22%) |
| RAM (за 1 ГБ) | {{ sku|RUB|mdb.cluster.mysql.v2.ram|month|string }}      | {{ sku|RUB|v1.commitment.selfcheckout.m6.mdb.mysql.ram.v2|month|string }} (-15%)      | {{ sku|RUB|v1.commitment.selfcheckout.y1.mdb.mysql.ram.v2|month|string }} (-22%)      |
| **Intel Ice Lake**                                                                                                                                                                                                                                       |
| 50% vCPU      | {{ sku|RUB|mdb.cluster.mysql.v3.cpu.c50|month|string }}  | −                                                                                     | −                                                                                     |
| 100% vCPU     | {{ sku|RUB|mdb.cluster.mysql.v3.cpu.c100|month|string }} | {{ sku|RUB|v1.commitment.selfcheckout.m6.mdb.mysql.cpu.c100.v3|month|string }} (-15%) | {{ sku|RUB|v1.commitment.selfcheckout.y1.mdb.mysql.cpu.c100.v3|month|string }} (-22%) |
| RAM (за 1 ГБ) | {{ sku|RUB|mdb.cluster.mysql.v3.ram|month|string }}      | {{ sku|RUB|v1.commitment.selfcheckout.m6.mdb.mysql.ram.v3|month|string }} (-15%)      | {{ sku|RUB|v1.commitment.selfcheckout.y1.mdb.mysql.ram.v3|month|string }} (-22%)      |
| **Intel Ice Lake (Compute Optimized)** |
| 100% vCPU | {{ sku|RUB|mdb.cluster.mysql.highfreq-v3.cpu.c100|month|string }} | − | − |
| RAM (за 1 ГБ) | {{ sku|RUB|mdb.cluster.mysql.highfreq-v3.ram|month|string }} | − | − |


