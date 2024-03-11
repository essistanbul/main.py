import sqlite3
import os
from bing_image_downloader.downloader import download
from PIL import Image

klasor_adi = "resim_klasoru"
db_adi = "resimler.sqlite3"

if not os.path.exists(klasor_adi):
    os.makedirs(klasor_adi)

db_yolu = os.path.join(klasor_adi, db_adi)
conn = sqlite3.connect(db_yolu)
cursor = conn.cursor()

cursor.execute("CREATE TABLE IF NOT EXISTS resimler (dosya_adi TEXT, dosya_yolu TEXT, genislik INTEGER, yukseklik INTEGER)")

def download_images_with_size_criteria(search_term, limit, timeout, min_width, min_height, max_width, max_height):
    
    print(f"{search_term} terimiyle ilgili resimler indiriliyor... Lütfen bekleyin.")
    downloader_args = {
        "limit": limit,
        "output_dir": "indirilen_resimler",
        "adult_filter_off": True,
        "force_replace": False,
        "timeout": timeout,
        "verbose": True,
    }
    download(search_term, **downloader_args)
    read_image_info_and_insert_into_db(os.path.join("indirilen_resimler", search_term), min_width, min_height, max_width, max_height)

def read_image_info_and_insert_into_db(folder_path, min_width, min_height, max_width, max_height):
    for root, _, files in os.walk(folder_path):
        for file in files:
            file_path = os.path.join(root, file)
            img = Image.open(file_path)
            if min_width <= img.width <= max_width and min_height <= img.height <= max_height:
                dosya_adi = file
                dosya_yolu = file_path  # Değiştirildi: dosya_yolu file_path olarak güncellendi.
                genislik = img.width
                yukseklik = img.height
                try:
                    cursor.execute("INSERT INTO resimler (dosya_adi, dosya_yolu, genislik, yukseklik) VALUES (?, ?, ?, ?)",
                                   (dosya_adi, dosya_yolu, genislik, yukseklik))
                    conn.commit()
                    print(f"{dosya_adi} adlı resim veritabanına kaydedildi.")
                except sqlite3.Error as e:
                    print("Veritabanı hatası:", e)
            else:
                print(f"{min_width}x{min_height} boyutlarında uygun resim bulunamadı.")
                print(f"{max_width}x{max_height}")
    print("Resim bilgileri veritabanına kaydedildi.")

name = input("Lütfen arama terimini giriniz: ")
limit = int(input("İndirilecek resim limitini giriniz: "))
timeout = int(input("İstek zaman aşımını giriniz (saniye cinsinden): "))
width = int(input("Minimum genişlik değerini giriniz: "))
height = int(input("Minimum yükseklik değerini giriniz: "))
max_width = int(input("Max genişlik değerini giriniz: "))
max_height = int(input("Max yükseklik değerini giriniz: "))
download_images_with_size_criteria(name, limit, timeout, width, height, max_width, max_height)
conn.close()

print("Resim bilgileri veritabanına kaydedildi.")
