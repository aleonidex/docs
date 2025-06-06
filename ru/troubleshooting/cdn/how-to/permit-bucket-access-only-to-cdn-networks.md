# Как ограничить доступ к бакету {{ objstorage-name }} диапазоном IP-адресов, принадлежащих {{ cdn-name }}


## Описание задачи {#case-description}

Необходимо, чтобы контент статического сайта или SPA, размещенного в {{ objstorage-full-name }}, был доступен только для {{ cdn-full-name }}.

## Решение {#case-resolution}

Логика размещения статических сайтов в {{ objstorage-name }} предусматривает, что содержимое бакета должно быть доступно для всех IP-адресов и подсетей, которые запрашивают контент с сайта.

Инфраструктура {{ cdn-full-name }} кеширует содержимое бакета с сайтом на своих серверах, однако в консоли управления пока отсутствует возможность настроить доступ к бакету исключительно из сетей CDN. 

Для реализации этого сценария необходимо ограничить публичный доступ к бакету по IP-адресам, вручную разрешив подключения только из подсетей {{ cdn-full-name }}, запрещая доступ к содержимому бакета для всех остальных IP-адресов. В этом случае необходимо настроить [политику доступа для бакета](../../../storage/concepts/policy.md), в которой будут указаны диапазоны IP-адресов, которые используются сервисом {{ cdn-full-name }}.

Список задействованных сервисом IP-адресов и подсетей можно получить по следующим ссылкам:

* [Список публичных IP-адресов провайдера EdgeЦентр](https://api.edgecenter.ru/cdn/public_ips_list);
* [Список подсетей провайдера EdgeЦентр c указанием CIDR](https://api.edgecenter.ru/cdn/public_net_list).

{% note info %}

Следует иметь в виду, что список этих адресов может периодически изменяться: в него могут быть включены новые или удалены более незадействованные IP-адреса/подсети.

Рекомендуем время от времени сверять список подсетей в созданной вами политике доступа для бакета с актуальным списком адресов.

{% endnote %}

Этот список адресов предоставляет в API наш партнер по CDN, [компания EdgeЦентр](https://apidocs.edgecenter.ru/cdn#tag/Networks-and-IP-addresses).


Примените политику из указанного ниже примера [по этой инструкции](../../../storage/operations/buckets/policy.md#apply-policy).
Перед применением политики на основе приложенного примера следует изменить `<имя бакета>` на наименование вашего бакета в {{ objstorage-name }}.

{% cut "Пример конфигурации" %}

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "103.242.74.224/29"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "185.211.231.192/29"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "5.188.132.0/28"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "195.34.58.164/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "217.118.183.248/30"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "178.176.145.176/28"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "194.67.48.24/30"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "176.56.179.176/29"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "98.158.98.226/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "87.239.191.78/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "195.3.244.188/30"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "92.223.123.96/27"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "138.204.14.150/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "81.211.44.154/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "220.158.132.70/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "92.223.108.0/27"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "194.186.66.252/30"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "5.188.121.128/25"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "178.176.153.0/24"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "178.176.145.96/27"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "5.101.68.0/27"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "91.243.83.0/26"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "31.173.147.2/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "92.223.122.160/27"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "178.176.131.224/28"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "195.239.135.56/30"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "178.176.145.0/27"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "92.223.110.0/27"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "87.229.240.64/30"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "5.188.7.0/26"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "178.176.131.128/26"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "92.50.131.190/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "109.200.129.16/29"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "194.154.79.64/30"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "92.223.123.0/26"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "92.223.64.0/28"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "185.101.137.0/28"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "92.223.114.0/26"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
         "aws:sourceip": "178.176.145.64/27"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
         "aws:sourceip": "195.239.185.56/30"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
         "aws:sourceip": "178.176.145.128/28"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
         "aws:sourceip": "178.176.145.144/28"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
          "aws:sourceip": "178.176.145.0/26"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
         "aws:sourceip": "5.8.43.0/28"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
         "aws:sourceip": "185.228.134.96/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
       "IpAddress": {
        "aws:sourceip": "5.188.121.128/26"
        }
      }
    },
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
      "IpAddress": {
      "aws:sourceip": "62.141.95.96/30"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
       "IpAddress": {
        "aws:sourceip": "92.223.43.0/26"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
       "IpAddress": {
        "aws:sourceip": "185.11.76.52/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
           "aws:sourceip": "178.176.131.0/26"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
            "aws:sourceip": "178.176.145.160/28"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
            "aws:sourceip": "31.173.147.0/31"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
            "aws:sourceip": "178.176.131.192/27"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
      "IpAddress": {
            "aws:sourceip": "178.176.131.240/28"
        }
       }
     },
     {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<имя бакета>/*",
      "Condition": {
        "IpAddress": {
            "aws:sourceip": "178.176.131.64/26"
        }
      }
    }
  ]
}

```
{% endcut %}
