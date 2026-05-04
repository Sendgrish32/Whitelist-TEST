import requests
import socket
import ssl
import time
from urllib.parse import urlparse
allowed_sites = {
    "https://yandex.ru": "Яндекс",
    "https://vk.com": "ВКонтакте",
    "https://max.ru": "MAX",
    "https://www.gosuslugi.ru": "Госуслуги"
}
blocked_sites = {
    "https://www.google.com": "Google",
    "https://www.youtube.com": "YouTube",
    "https://www.wikipedia.org": "Wikipedia",
    "https://www.reddit.com": "Reddit",
    "https://www.github.com": "GitHub",
    "https://www.instagram.com": "Instagram",    
    "https://x.com": "X (Twitter)",
    "https://www.netflix.com": "Netflix",
    "https://www.twitch.tv": "Twitch",    
    "https://www.microsoft.com": "Microsoft",
    "https://www.apple.com": "Apple",
    "https://www.discord.com": "Discord",
    "https://www.zoom.us": "Zoom",
    "https://www.spotify.com": "Spotify"
}
def check_site(url, timeout=10):
    try:
        start_time = time.time()
        try:
            response = requests.head(url, timeout=timeout, allow_redirects=True, verify=False)
            if response.status_code >= 400:
                response = requests.get(url, timeout=timeout, allow_redirects=True, verify=False, stream=True)
                response.close()
        except:
            response = requests.get(url, timeout=timeout, allow_redirects=True, verify=False, stream=True)
            response.close()
            
        elapsed = time.time() - start_time
        is_available = 200 <= response.status_code < 400
        
        return is_available, response.status_code, round(elapsed, 2), None
        
    except requests.exceptions.Timeout:
        return False, None, timeout, "Timeout"
    except requests.exceptions.ConnectionError:
        return False, None, None, "Connection Error (вероятно, заблокирован)"
    except requests.exceptions.SSLError:
        return False, None, None, "SSL Error"
    except Exception as e:
        return False, None, None, str(e)[:50]

def main():
    print("=" * 70)
    print("ПРОВЕРКА БЕЛЫХ СПИСКОВ НА МОБИЛЬНОМ ИНТЕРНЕТЕ РФ")
    print("Режим: ДОЛЖНЫ работать только Яндекс, ВК, MAX, Госуслуги")
    print("=" * 70)
    
    print("\n📱 ПОДКЛЮЧЕНИЕ: Убедитесь, что вы используете МОБИЛЬНЫЙ ИНТЕРНЕТ!")
    print("   (WiFi отключите для чистоты эксперимента)\n")
    
    input("Нажмите ENTER, чтобы начать проверку...")
    print("\n" + "=" * 70)
    
    results = {
        "allowed": {"ok": 0, "total": 0, "details": []},
        "blocked": {"blocked": 0, "total": 0, "details": []},
        "grey": {"details": []}
    }
    print("\n✅ ПРОВЕРКА РАЗРЕШЁННЫХ САЙТОВ:")
    print("-" * 70)
    
    for url, name in allowed_sites.items():
        print(f"  {name} ({url})... ", end="", flush=True)
        available, status, elapsed, error = check_site(url)
        
        results["allowed"]["total"] += 1
        
        if available:
            print(f"✅ ДОСТУПЕН (статус: {status}, {elapsed}с)")
            results["allowed"]["ok"] += 1
            results["allowed"]["details"].append(f"{name}: ✅ ДОСТУПЕН")
        else:
            print(f"❌ НЕДОСТУПЕН - {error if error else 'Нет ответа'}")
            results["allowed"]["details"].append(f"{name}: ❌ НЕДОСТУПЕН")
    
    print("\n\n🚫 ПРОВЕРКА ЗАПРЕЩЁННЫХ САЙТОВ:")
    print("-" * 70)
    
    for url, name in blocked_sites.items():
        print(f"  {name} ({url})... ", end="", flush=True)
        available, status, elapsed, error = check_site(url)
        
        results["blocked"]["total"] += 1
        
        if not available:
            print(f"❌ ЗАБЛОКИРОВАН - {error if error else 'Нет доступа'}")
            results["blocked"]["blocked"] += 1
            results["blocked"]["details"].append(f"{name}: ✅ ЗАБЛОКИРОВАН")
        else:
            print(f"✅ ДОСТУПЕН (статус: {status}, {elapsed}с)")
            results["blocked"]["details"].append(f"{name}: ❌ ДОСТУПЕН")
    
    print("\n" + "=" * 70)
    print("РЕЗУЛЬТАТЫ ПРОВЕРКИ:")
    print("=" * 70)
    
    allowed_success_rate = (results["allowed"]["ok"] / results["allowed"]["total"]) * 100
    blocked_block_rate = (results["blocked"]["blocked"] / results["blocked"]["total"]) * 100
    
    print(f"\n📊 СТАТИСТИКА:")
    print(f"  • Разрешённые сайты доступны: {results['allowed']['ok']}/{results['allowed']['total']} ({allowed_success_rate:.0f}%)")
    print(f"  • Запрещённые сайты заблокированы: {results['blocked']['blocked']}/{results['blocked']['total']} ({blocked_block_rate:.0f}%)")
    
    print(f"\n🔍 ВЕРДИКТ:")
    
    # Если разрешённые работают, а запрещённые НЕ работают - белые списки активны
    if results["allowed"]["ok"] == results["allowed"]["total"] and results["blocked"]["blocked"] == results["blocked"]["total"]:
        print("  ❌ БЕЛЫЕ СПИСКИ, СКОРЕЕ ВСЕГО, АКТИВНЫ")
        print("  Доступны только разрешённые сайты (Яндекс, ВК, MAX, Госуслуги)")
    elif results["allowed"]["ok"] < results["allowed"]["total"]:
        print("  ❌ ПРОБЛЕМЫ С ДОСТУПОМ К РАЗРЕШЁННЫМ САЙТАМ!")
        print("  Возможно, проблемы с интернет-соединением или DNS")
    elif results["blocked"]["blocked"] < results["blocked"]["total"]:
        print("  ✅ БЕЛЫЕ СПИСКИ НЕ АКТИВНЫ")
        print("  Запрещённые сайты доступны, значит фильтрации нет или включён только TSPU")
        if results["blocked"]["blocked"] == 0:
            print("  Все проверенные иностранные сайты работают - белых списков нет!")
    else:
        print("  🟡 СМЕШАННЫЙ РЕЗУЛЬТАТ")
        print("  Возможно, работает частичная фильтрация или проблемы с сетью")
    
    print("\n" + "=" * 70)
    print("ВАЖНО:")
    print("• Тест проведён через HTTP/HTTPS, не через ping")
    print("• Результат зависит от конкретного оператора и региона")
    print("• При активных белых списках ЗАПРЕЩЁННЫЕ сайты НЕ должны открываться")
    print("=" * 70)

if __name__ == "__main__":
    # Отключаем предупреждения о SSL (для чистоты вывода)
    import urllib3
    urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
    
    main()
