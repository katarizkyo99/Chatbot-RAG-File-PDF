Markdown

# Chatbot RAG - Football Player Stats

Sebuah proyek Chatbot berbasis **Retrieval-Augmented Generation (RAG)** yang dirancang untuk menjawab pertanyaan dan memberikan informasi spesifik berdasarkan dokumen PDF data statistik pemain sepak bola.

Struktur Repositori

Chatbot RAG/
│
├── .env                                  # File konfigurasi untuk environment variables (API Keys, dll.)
├── main.py                               # Skrip utama aplikasi (Backend/Logika Chatbot)
├── DB/
│   └── DB_Football Player Stats.pdf      # Dokumen sumber (knowledge base) untuk sistem RAG
└── templates/
    └── chat.html                         # Antarmuka pengguna (Frontend/Web UI)

Cara Instalasi & Penggunaan
1. Clone Repositori
git clone https://github.com/katarizkyo99/Chatbot-RAG-File-PDF.git

2. Konfigurasi Environment
API_KEY=your_api_key_here

3. Jalankan Aplikasi
Eksekusi skrip utama backend untuk menyalakan server:
python main.py

5. Akses Chatbot
Buka browser Anda dan akses URL lokal yang diberikan oleh terminal untuk mulai berinteraksi dengan chatbot.
