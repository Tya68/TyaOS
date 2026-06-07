# TyaOS
This is the official artifical intelligence (AI) of the TyaOS
Every creator can change code and settnigs (You need allow for the be creator of the TyaOS)
import os
import random
import datetime
import tkinter as tk
from tkinter import scrolledtext
import requests
from bs4 import BeautifulSoup

LOG_FILE = "bot_loglari.txt"

def log_yaz(gonderen, mesaj):
    """Botun ve kullanıcının konuşmalarını bir metin dosyasına kaydeder."""
    zaman = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    try:
        with open(LOG_FILE, "a", encoding="utf-8") as f:
            f.write(f"[{zaman}] {gonderen}: {mesaj}\n")
    except Exception:
        pass # Log yazma hatası programı durdurmasın

def hava_durumu_al(sehir):
    """Belirtilen şehrin hava durumunu internetten anlık çeker."""
    try:
        # Türkçe karakterleri basitçe düzenleme
        sehir = sehir.replace("i", "i").replace("ı", "i").lower()
        url = f"https://www.ntv.com.tr/hava-durumu/{sehir}"
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'}
        response = requests.get(url, headers=headers, timeout=5)
        
        if response.status_code == 200:
            soup = BeautifulSoup(response.text, 'html.parser')
            derece = soup.find("div", {"class": "hava-durumu-detail-status-degree"}).text.strip()
            durum = soup.find("div", {"class": "hava-durumu-detail-status-text"}).text.strip()
            return f"{sehir.capitalize()} için hava şu an {derece}°C ve {durum}."
        return f"{sehir.capitalize()} şehrinin hava durumu bilgisini şu an çekemedim. İsmi doğru yazdığınızdan emin olun."
    except Exception:
        return "Hava durumu servisine bağlanırken bir hata oluştu."

def internette_ara(sorgu):
    """Kullanıcının merak ettiği bir konuyu internette aratıp ilk sonucu özetler."""
    try:
        url = f"https://html.duckduckgo.com/html/?q={sorgu}"
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'}
        response = requests.get(url, headers=headers, timeout=5)
        if response.status_code == 200:
            soup = BeautifulSoup(response.text, 'html.parser')
            sonuclar = soup.find_all("a", {"class": "result__snippet"})
            if sonuclar:
                return f"İnternette bulduğum özet bilgi: {sonuclar[0].text.strip()}"
        return "Bununla ilgili internette açık bir sonuç bulamadım."
    except Exception:
        return "İnternet araması şu an gerçekleştirilemiyor."

class BotBeyni:
    def __init__(self):
        # Kısa süreli hafıza (Mevcut sohbet oturumu için)
        self.hafiza = []
        
        # Gelişmiş kelime eşleştirme veritabanı
        self.sabit_cevaplar = {
            "selam": ["Selam! Nasıl yardımcı olabilirim?", "Merhaba! Bugün harika bir gün.", "Ooo merhaba, hoş geldin!"],
            "merhaba": ["Merhaba! Nasıl gidiyor?", "Selamlar, bugün ne yapıyoruz?", "Merhaba, size nasıl yardım edebilirim?"],
            "nasılsın": ["Harikayım! Kodlarım tıkır tıkır çalışıyor. Sen nasılsın?", "Bir yapay zeka olarak her zaman hazırım!", "İyiyim, seni sormalı?"],
            "kimsin": ["Ben senin için geliştirilmiş özel bir masaüstü asistanıyım.", "Henüz resmi bir adım yok ama görevim sana yardım etmek."],
            "teşekkür": ["Rica ederim, ne demek!", "Görevimiz efendim.", "Her zaman!"],
            "ne yapabilirsin": ["Seninle sohbet edebilirim, matematik çözebilirim, hava durumunu söyleyebilirim veya 'ara: konu' yazarsan internette arama yapabilirim!"]
        }

    def MatematikHesapla(self, metin):
        """Metin içerisindeki matematiksel denklemleri çözer."""
        temiz_metin = "".join(c for c in metin if c in "0123456789+-*/()")
        if temiz_metin:
            try:
                return f"Matematiksel işlemin sonucu: {eval(temiz_metin)}"
            except:
                return None
        return None

    def yanit_uret(self, kullanici_girdisi):
        girdi_guncel = kullanici_girdisi.lower().strip()
        log_yaz("Kullanıcı", kullanici_girdisi)
        
        self.hafiza.append(f"Kullanıcı: {kullanici_girdisi}")
        bot_yaniti = ""
        
        # 1. Aşama: Hava Durumu Kontrolü
        if "hava durumu" in girdi_guncel or "hava nasıl" in girdi_guncel:
            kelimeler = girdi_guncel.split()
            sehir = "istanbul"  # Varsayılan şehir
            for k in kelimeler:
                if k not in ["hava", "durumu", "nasıl", "bugün", "yarın", "in", "da", "de", "bana"]:
                    sehir = k
                    break
            bot_yaniti = hava_durumu_al(sehir)
            
        # 2. Aşama: İnternet Arama Kontrolü (Örn: ara: yapay zeka nedir)
        elif "ara:" in girdi_guncel or girdi_guncel.startswith("nedir "):
            sorgu = girdi_guncel.replace("ara:", "").replace("nedir", "").strip()
            bot_yaniti = internette_ara(sorgu)
            
        # 3. Aşama: Matematiksel İşlem Kontrolü
        elif any(c in girdi_guncel for c in "+-*/") and any(c.isdigit() for c in girdi_guncel):
            mat_sonuc = self.MatematikHesapla(girdi_guncel)
            bot_yaniti = mat_sonuc if mat_sonuc else "Matematik işlemini hesaplayamadım, formatı kontrol et."

        # 4. Aşama: Sabit Kelime Eşleşmeleri
        else:
            cevap_bulundu = False
            for anahtar, cevap_listesi in self.sabit_cevaplar.items():
                if anahtar in girdi_guncel:
                    bot_yaniti = random.choice(cevap_listesi)
                    cevap_bulundu = True
                    break
            
            # Eğer hiçbir şeye uymuyorsa genel yanıt
            if not cevap_bulundu:
                bot_yaniti = "Bunu henüz tam anlayamadım. Kendimi geliştirmem için bana yeni kelimeler öğretmelisin! İnternette aratmak istersen 'ara: konu' şeklinde yazabilirsin."

        # Botun yanıtını hafızaya ve loglara işle
        self.hafiza.append(f"Bot: {bot_yaniti}")
        log_yaz("Bot", bot_yaniti)
        
        # Hafızanın aşırı şişmesini engelle (Son 20 mesajı tut)
        if len(self.hafiza) > 20:
            self.hafiza = self.hafiza[-20:]
            
        return bot_yaniti

class BotArayuzu:
    def __init__(self, pencere):
        self.pencere = pencere
        self.pencere.title("Gelişmiş Yapay Zeka Asistanı v1.0")
        self.pencere.geometry("500x600")
        self.pencere.configure(bg="#2c3e50") # Koyu modern tema arka planı
        
        # Beyin modülünü arayüze entegre ediyoruz
        self.beyin = BotBeyni()
        
        self.ArayuzOlustur()
        
    def ArayuzOlustur(self):
        # Üst Kısım: Sohbet Geçmişi Alanı
        self.sohbet_alani = scrolledtext.ScrolledText(
            self.pencere, 
            wrap=tk.WORD, 
            bg="#34495e", 
            fg="white", 
            font=("Arial", 11)
        )
        self.sohbet_alani.pack(padx=10, pady=10, fill=tk.BOTH, expand=True)
        self.sohbet_alani.config(state=tk.DISABLED) # Kullanıcı buradaki metinleri silemesin
        
        # Yazı renk etiketleri (Kullanıcı mavi, bot yeşil görünsün)
        self.sohbet_alani.tag_config("Sen", foreground="#3498db", font=("Arial", 11, "bold"))
        self.sohbet_alani.tag_config("Bot", foreground="#2ecc71", font=("Arial", 11, "bold"))

        # Alt Kısım: Mesaj Giriş Paneli
        giris_cercevesi = tk.Frame(self.pencere, bg="#2c3e50")
        giris_cercevesi.pack(padx=10, pady=10, fill=tk.X, side=tk.BOTTOM)
        
        # Yazı Yazma Kutusu
        self.giris_kutusu = tk.Entry(
            giris_cercevesi, 
            font=("Arial", 12), 
            bg="#ecf0f1", 
            fg="#2c3e50"
        )
        self.giris_kutusu.pack(side=tk.LEFT, fill=tk.X, expand=True, ipady=8)
        self.giris_kutusu.bind("<Return>", self.MesajGonderEtkinligi) # Enter'a basınca gönderir
        
        # Gönder Butonu
        gonder_butonu = tk.Button(
            giris_cercevesi, 
            text="Gönder", 
            font=("Arial", 10, "bold"), 
            bg="#2ecc71", 
            fg="white", 
            command=self.MesajGonder,
            activebackground="#27ae60"
        )
        gonder_butonu.pack(side=tk.RIGHT, padx=5, ipady=5, ipadx=10)
        
        # İlk karşılama mesajı
        self.EkranaYaz("Bot", "Sistem aktif. Yapay Zeka Asistanı başlatıldı! Merhaba, size nasıl yardımcı olabilirim?")

    def MesajGonderEtkinligi(self, event):
        """Klavyeden Enter tuşuna basıldığında çalışır."""
        self.MesajGonder()

    def MesajGonder(self):
        """Kullanıcının yazdığı mesajı alır, ekrana yazar ve bota iletir."""
        kullanici_mesaji = self.giris_kutusu.get().strip()
        if not kullanici_mesaji:
            return
            
        self.EkranaYaz("Sen", kullanici_mesaji)
        self.giris_kutusu.delete(0, tk.END) # Giriş kutusunu temizle
        
        # Beyin fonksiyonunu çağırıp bot cevabını alıyoruz
        bot_cevati = self.beyin.yanit_uret(kullanici_mesaji)
        self.EkranaYaz("Bot", bot_cevati)

    def EkranaYaz(self, gonderen, mesaj):
        """Sohbet ekranına mesajları biçimlendirerek ekler."""
        self.sohbet_alani.config(state=tk.NORMAL)
        
        if gonderen == "Sen":
            self.sohbet_alani.insert(tk.END, "\nSiz: ", "Sen")
            self.sohbet_alani.insert(tk.END, f"{mesaj}\n")
        else:
            self.sohbet_alani.insert(tk.END, "\nAsistan: ", "Bot")
            self.sohbet_alani.insert(tk.END, f"{mesaj}\n")
            
        self.sohbet_alani.config(state=tk.DISABLED)
        self.sohbet_alani.yview(tk.END) # Sohbeti otomatik olarak en aşağı kaydır


if __name__ == "__main__":
    # Ana pencere nesnesi oluşturuluyor
    root = tk.Tk()
    
    # Uygulama başlatılıyor
    app = BotArayuzu(root)
    
    # Pencerenin ekranda kalmasını sağlayan döngü
    root.mainloop()
