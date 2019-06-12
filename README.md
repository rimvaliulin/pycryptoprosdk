# pycryptoprosdk
Библиотека для работы с Cryptopro CSP в python

## Установка
1. Установить КриптоПро CSP.
2. Установить пакеты lsb-cprocsp-devel-.noarch.rpm и cprocsp-pki-amd64-cades.rpm из состава КриптоПро ЭЦП SDK.
3. При необходимости, создать симлинк:
```
ln -s /opt/cprocsp/lib/amd64/libcades.so.2.0.0 /opt/cprocsp/lib/amd64/libcades.so
```
Пример установки пакетов можно посмотреть в [pycryptoprosdk/compose/Dockerfile](https://github.com/Keyintegrity/pycryptoprosdk/blob/master/compose/Dockerfile).

4. Установить pycryptoprosdk:
```
python setup.py install
```

## Примеры использования
```python
from base64 import b64encode
from pycryptoprosdk import CryptoProSDK


sdk = CryptoProSDK()


# Создание и проверка отсоединенной подписи:
content = b64encode(b'test content')
cert = sdk.get_cert_by_subject('MY', 'Ivan')
signature = sdk.sign(content, cert.thumbprint, 'MY', detached=True)

result = sdk.verify_detached(content, signature)

# статус проверки:
result.verification_status

0: Успешная проверка подписи.
1: Отсутствуют или имеют неправильный формат атрибуты со ссылками и значениями доказательств подлинности.
2: Сертификат, на ключе которого было подписано сообщение, не найден.
3: В сообщении не найден действительный штамп времени на подпись.
4: Значения ссылок на доказательства подлинности и сами доказательства, вложенные в сообщение, не соответствуют друг другу.
5: Не удалось построить цепочку для сертификата, на ключе которого подписано сообщение.
6: Ошибка проверки конечного сертификата на отзыв.
7: Ошибка проверки сертификата цепочки на отзыв.
8: Сообщение содержит неверную подпись.
9: В сообщении не найден действительный штамп времени на доказательства подлинности подписи.
10: Значение подписанного атрибута content-type не совпадает со значением, указанным в поле encapContentInfo.eContentType.

# сертификат подписанта:
result.cert.as_dict()


# создание хэша файла алгоритмом ГОСТ Р 34.11-94:
with open('doc.txt'), 'rb') as f:
    content = f.read()
h = sdk.create_hash(content, alg='CALG_GR3411')


# поиск сертификата в хранилище MY по отпечатку:
cert = sdk.get_cert_by_thumbprint('MY', '046255290b0eb1cdd1797d9ab8c81f699e3687f3')


# поиск сертификата по имени:
cert = sdk.get_cert_by_subject('MY', 'CRYPTO-PRO Test Center 2')


# установка сертификата в хранилище MY:
with open('certificate.cer'), 'rb') as f:
    cert_content = f.read()
sdk.install_certificate('MY', b64encode(cert_content))


# удаление сертификата из хранилища MY по отпечатку:
sdk.delete_certificate('MY', '9e78a331020e528c046ffd57704a21b7d2241cb3')


# извлечение сертификата подписанта из подписи:
with open('signature.sig', 'rb') as f:
    signature_content = f.read()
cert = sdk.get_signer_cert_from_signature(signature_content)
```
