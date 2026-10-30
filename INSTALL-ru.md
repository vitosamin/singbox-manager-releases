# Установка sing-box manager

Панель управления sing-box на роутере: серверы, правила обхода по гео-тегам и
своим спискам, приоритеты, адаптивная маршрутизация, DNS (включая DoH).
Работает на Keenetic (Entware) и OpenWrt.

Пакет содержит шесть архитектур (`mipsle`, `mips`, `armv7`, `arm64`, `amd64`,
`riscv64`), установщик и этот файл. Установщик сам определяет архитектуру,
проверяя бинарник **запуском**, а не по названию — на старых Keenetic `mips` и
`mipsle` путаются, и по названию легко поставить нерабочий.

## Постоянные ссылки

- **Установщик (все платформы и архитектуры):**
  https://github.com/vitosamin/singbox-manager-releases/releases/latest/download/singbox-manager-installer.tar.gz
- **Эта инструкция:**
  https://github.com/vitosamin/singbox-manager-releases/blob/main/INSTALL-ru.md

Ссылки не меняются от версии к версии: `latest` всегда отдаёт свежий пакет.

## Что нужно

1. Keenetic с установленным Entware **или** OpenWrt (24 — opkg, 25 — apk).
2. Свободный порт **9091**.
3. Место под пакет: ~150 МБ на Keenetic (`/opt/tmp`), на OpenWrt хватит `/tmp`,
   если в нём есть 50 МБ — иначе см. «Мало места».

Установщик сам ставит **sing-box 1.14.2** — официальный архив SagerNet под
архитектуру роутера (mips/mipsle — softfloat). Пакет из `opkg`/`apk` для
sing-box не используется: на OpenWrt 24 там 1.13, и панель с таким конфигом
ругается на неизвестные поля. Сам пакетный менеджер нужен только для `wget`
и `ca-bundle` (`opkg update` / `apk update` установщик делает сам).

## Keenetic

```sh
cd /opt/tmp
wget -O sbm.tar.gz https://github.com/vitosamin/singbox-manager-releases/releases/latest/download/singbox-manager-installer.tar.gz
tar -xzf sbm.tar.gz
cd singbox-manager-*
sh install.sh
```

Ставится в `/opt/bin/singbox-manager`, автозапуск —
`/opt/etc/init.d/S99singbox-manager`, состояние — `/opt/etc/sing-box/state.json`.

## OpenWrt

```sh
cd /tmp
wget -O sbm.tar.gz https://github.com/vitosamin/singbox-manager-releases/releases/latest/download/singbox-manager-installer.tar.gz
tar -xzf sbm.tar.gz && cd singbox-manager-* && sh install.sh
```

Ставится в `/usr/bin/singbox-manager`, сервис — `/etc/init.d/singbox-manager`
(через procd, с автоперезапуском), состояние — `/etc/sing-box/state.json`.

### Мало места (64 МБ RAM)

43 МБ в `/tmp` не влезут. Распакуйте архив на компьютере и залейте только нужный
бинарник (`uname -m` на роутере подскажет: `armv7`, `aarch64`, `mips`, `mipsel`,
`x86_64`, `riscv64`):

```sh
scp singbox-manager-*/singbox-manager-armv7 root@192.168.1.1:/tmp/singbox-manager
ssh root@192.168.1.1 'chmod 755 /tmp/singbox-manager && mv /tmp/singbox-manager /usr/bin/ && /etc/init.d/singbox-manager restart'
```

## Вопрос про DNS при установке

Установщик спросит, чем решать имена:

| Вариант | Что произойдёт |
|---|---|
| **1. DoH (Quad9)** | панель включит DNS-over-HTTPS через Quad9 — по умолчанию; провайдера потом можно сменить в панели |
| **2. Свой DNS в локальной сети** | если у вас уже работает AdGuard Home или dnsmasq — мы **ничего не ставим**, панель просто переключится на него |
| **3. Не трогать** | оставить как настроено |

Ответ можно задать заранее, без вопроса: `DNS_MODE=doh`, `DNS_MODE=own` или
`DNS_MODE=none` в окружении установщика. Выбор применяется **один раз** при
первом запуске панели; потом его не перебивает.

**Провайдера DoH можно выбрать.** По умолчанию — Quad9. Сменить: панель →
«Настройки» → «DNS-серверы» → список провайдеров и кнопка **DoH**. Доступны
Quad9, Cloudflare, Google, AdGuard DNS, DNS.SB; рядом — кнопка
**«Свой AdGuard Home»**, если резолвер уже работает на роутере.

Сторонние пакеты (`https-dns-proxy`, `dnsproxy`) не нужны — DoH умеет сам
sing-box.

**DNS всей сети.** «Настройки» → «DNS для всей сети»: панель показывает, какой
DNS сейчас спрашивают телефоны, и переключает его одной кнопкой — **«Yandex
DNS»** (77.88.8.8, не фильтрует соцсети) или **«Как от провайдера»**. Работает
на OpenWrt (правит dnsmasq). Если DNS обслуживает AdGuard Home, переключайте
в нём — панель об этом предупредит.

**Резерв DNS.** Там же, в «DNS-серверах», можно выбрать резервный сервер:
если основной три проверки подряд не ответит, панель переключится на резерв,
а после двух успехов вернёт обратно. Резерв должен быть доступен без туннеля
(AdGuard Home, DNS провайдера).

## Первый вход

Откройте `http://<адрес-роутера>:9091`, задайте пароль.

Дальше: «Серверы» → добавить сервер → «Правила обхода» → выбрать гео-теги или
применить шаблон → кнопка «Применить» наверху.

## Обновления

Внизу меню — «Проверить обновление». Панель сама проверяет публичную страницу
релизов и ставит новую версию. Ключи и регистрация не нужны: в сборке нет
никаких токенов.

`https://github.com/vitosamin/singbox-manager-releases/releases/latest`

## Поверх уже установленной версии

Если панель уже стоит — просто запустите `sh install.sh` из распакованного
пакета: бинарник заменится, сервис перезапустится, **`state.json` не
трогается** (серверы, правила, списки, пароль остаются).

Сначала полезно понять, что запущено:

```sh
ps | grep "[s]ingbox-manager"
readlink -f /proc/$(pidof singbox-manager)/exe
```

Если путь не `/opt/bin/singbox-manager` (Keenetic) или не
`/usr/bin/singbox-manager` (OpenWrt) — старый экземпляр запускался иначе.
Убейте его (`kill $(pidof singbox-manager)`), поставьте пакет и проверьте, что
старый автозапуск не остался, иначе после перезагрузки поднимется старая копия
и займёт порт:

```sh
grep -rl singbox-manager /opt/etc/init.d/ /etc/init.d/ 2>/dev/null
```

## Проверка, что всё живо

```sh
ps | grep -c "[s]ingbox-manager"                                  # 1
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:9091/   # 200
```

Не открывается панель — смотрите лог: Keenetic
`/opt/var/log/singbox-manager.log`, OpenWrt `logread | grep singbox-manager`.

## Удаление

```sh
# Keenetic
rm -f /opt/etc/init.d/S99singbox-manager /opt/bin/singbox-manager
rm -rf /opt/etc/sing-box

# OpenWrt
/etc/init.d/singbox-manager stop && /etc/init.d/singbox-manager disable
rm -f /etc/init.d/singbox-manager /usr/bin/singbox-manager
rm -rf /etc/sing-box
```

Учтите: удаление состояния стирает пароль, серверы и правила.
