import tkinter as tk
import random
import math

class BallAnimation:
    def __init__(self, root):
        self.root = root
        self.root.title("Ball Animation")
        self.root.geometry("820x650")
        self.root.configure(bg="lightgray")
        
        # Canvas boyutları
        self.canvas_width = 800
        self.canvas_height = 450
        
        # Top boyutları ve renkleri
        self.ball_sizes = [20, 35, 50]  # Küçük, Orta, Büyük
        self.ball_colors = ["red", "blue", "yellow"]
        
        # Toplar listesi
        self.balls = []
        self.animation_running = False
        self.speed_multiplier = 1.0
        self.selected_size_index = 0
        self.selected_color = None  # Seçili renk (başlangıçta yok)
        
        # Canvas oluştur - light gray arka plan
        self.canvas = tk.Canvas(
            root,
            width=self.canvas_width,
            height=self.canvas_height,
            bg="lightgray",
            highlightthickness=0
        )
        self.canvas.pack(pady=10)
        
        # Canvas'a tıklama event'i - hem Button-1 hem de ButtonRelease-1
        self.canvas.bind("<Button-1>", self.add_ball)
        self.canvas.bind("<ButtonRelease-1>", self.add_ball)
        # Canvas'a odak ver
        self.canvas.focus_set()
        
        # Kontrol paneli - beyaz/açık gri arka plan (görseldeki gibi)
        control_panel = tk.Frame(root, bg="white")
        control_panel.pack(pady=10, padx=10)
        
        # Üst satır: Renk seçici butonlar
        color_row = tk.Frame(control_panel, bg="white")
        color_row.pack(pady=5, anchor=tk.W)
        
        tk.Label(color_row, text="Renk Seçin:", bg="white", font=("Arial", 10)).pack(side=tk.LEFT, padx=5)
        
        # Renk isimleri ve yazı renkleri
        color_names = {"red": "Kırmızı", "blue": "Mavi", "yellow": "Sarı"}
        text_colors = {"red": "white", "blue": "white", "yellow": "black"}
        
        self.color_buttons = []
        for color in self.ball_colors:
            color_frame = tk.Frame(color_row, bg="white")
            color_frame.pack(side=tk.LEFT, padx=10)
            
            # Canvas kullanarak renkli buton oluştur
            color_canvas = tk.Canvas(
                color_frame,
                width=80,
                height=35,
                bg=color,
                highlightthickness=2,
                highlightbackground="gray",
                relief=tk.RAISED,
                borderwidth=2
            )
            color_canvas.pack()
            
            # Buton üzerine yazı ekle
            color_canvas.create_text(
                40, 17,
                text=color_names[color],
                fill=text_colors[color],
                font=("Arial", 9, "bold")
            )
            
            # Tıklama event'i
            def make_color_handler(c):
                def handler(event):
                    self.select_color(c)
                return handler
            
            color_canvas.bind("<Button-1>", make_color_handler(color))
            self.color_buttons.append((color, color_canvas))
        
        # İkinci satır: Boyut seçici daireler (3 adet, küçükten büyüğe)
        size_row = tk.Frame(control_panel, bg="white")
        size_row.pack(pady=3, anchor=tk.W)
        
        self.size_indicators = []
        for i, size in enumerate(self.ball_sizes):
            size_container = tk.Frame(size_row, bg="white")
            size_container.pack(side=tk.LEFT, padx=15)
            
            # Gri daire göstergesi
            indicator = tk.Canvas(
                size_container,
                width=size + 20,
                height=size + 20,
                bg="white",
                highlightthickness=0
            )
            indicator.pack()
            # Gri daire çiz
            indicator.create_oval(
                10, 10, size + 10, size + 10,
                fill="gray",
                outline="black",
                width=2
            )
            
            # Tıklama event'i
            def make_click_handler(idx):
                def handler(event):
                    self.select_size(idx)
                return handler
            
            indicator.bind("<Button-1>", make_click_handler(i))
            self.size_indicators.append(indicator)
        
        # İlk boyut seçili
        self.update_size_display()
        
        # Alt satır: Butonlar - görseldeki gibi
        button_row = tk.Frame(control_panel, bg="white")
        button_row.pack(pady=0, anchor=tk.W)
        
        # START butonu - kırmızı kare, metin altında (siyah)
        start_frame = tk.Frame(button_row, bg="white")
        start_frame.pack(side=tk.LEFT, padx=10, pady=0)
        self.start_btn = tk.Button(
            start_frame,
            bg="red",
            width=4,
            height=2,
            command=self.start_animation,
            relief=tk.RAISED,
            borderwidth=2
        )
        self.start_btn.pack()
        # Label ile yazı - kesinlikle görünür
        start_label = tk.Label(
            start_frame,
            text="START",
            bg="white",
            fg="black",
            font=("Arial", 13, "bold"),
            pady=3,
            padx=5
        )
        start_label.pack()
        
        # STOP butonu - mavi kare, metin altında (siyah)
        stop_frame = tk.Frame(button_row, bg="white")
        stop_frame.pack(side=tk.LEFT, padx=10, pady=0)
        self.stop_btn = tk.Button(
            stop_frame,
            bg="blue",
            width=4,
            height=2,
            command=self.stop_animation,
            relief=tk.RAISED,
            borderwidth=2
        )
        self.stop_btn.pack()
        # Label ile yazı - kesinlikle görünür
        stop_label = tk.Label(
            stop_frame,
            text="STOP",
            bg="white",
            fg="black",
            font=("Arial", 13, "bold"),
            pady=3,
            padx=5
        )
        stop_label.pack()
        
        # RESET butonu - sarı kare, metin altında (siyah)
        reset_frame = tk.Frame(button_row, bg="white")
        reset_frame.pack(side=tk.LEFT, padx=10, pady=0)
        self.reset_btn = tk.Button(
            reset_frame,
            bg="yellow",
            width=4,
            height=2,
            command=self.reset_animation,
            relief=tk.RAISED,
            borderwidth=2
        )
        self.reset_btn.pack()
        # Label ile yazı - kesinlikle görünür
        reset_label = tk.Label(
            reset_frame,
            text="RESET",
            bg="white",
            fg="black",
            font=("Arial", 13, "bold"),
            pady=3,
            padx=5
        )
        reset_label.pack()
        
        # Speed Up butonu - gri dikdörtgen, metin içinde (siyah)
        speed_frame = tk.Frame(button_row, bg="white")
        speed_frame.pack(side=tk.LEFT, padx=10, pady=0)
        self.speed_btn = tk.Button(
            speed_frame,
            text="Speed Up",
            bg="gray",
            fg="black",
            font=("Arial", 10, "bold"),
            width=12,
            height=2,
            command=self.speed_up,
            relief=tk.RAISED,
            borderwidth=2
        )
        self.speed_btn.pack()
    
    def select_color(self, color):
        """Renk seçimi"""
        self.selected_color = color
        # Seçili rengi vurgula
        for col, canvas in self.color_buttons:
            if col == color:
                canvas.config(highlightthickness=3, highlightbackground="black")
            else:
                canvas.config(highlightthickness=2, highlightbackground="gray")
        print(f"Renk seçildi: {color}")
    
    def select_size(self, index):
        """Boyut seçimi"""
        self.selected_size_index = index
        self.update_size_display()
    
    def update_size_display(self):
        """Boyut göstergelerini güncelle"""
        for i, indicator in enumerate(self.size_indicators):
            if i == self.selected_size_index:
                indicator.config(bg="#b0d0ff")  # Seçili olanı mavi yap
            else:
                indicator.config(bg="white")
    
    def add_ball(self, event):
        """Canvas'a top ekle - seçili renkte, farklı boyutlarda ve oval şekilde"""
        # Event'in canvas'tan geldiğinden emin ol
        if event.widget != self.canvas:
            return
        
        # Renk seçilmemişse uyarı ver
        if self.selected_color is None:
            print("Lütfen önce bir renk seçin!")
            return
        
        # Seçili boyutu kullan
        size = self.ball_sizes[self.selected_size_index]
        
        # Oval şekil için width ve height farklı
        width = size
        height = size * 0.7  # Oval yapmak için height'i küçült
        
        radius_x = width / 2
        radius_y = height / 2
        
        # Rastgele konum (kenarlardan uzakta)
        x = random.uniform(radius_x, self.canvas_width - radius_x)
        y = random.uniform(radius_y, self.canvas_height - radius_y)
        
        # Seçili renk kullan
        color = self.selected_color
        
        # Başlangıç hızı (START'a basılınca verilecek)
        velocity_x = 0
        velocity_y = 0
        
        # Eğer animasyon çalışıyorsa hemen rastgele hız ver
        if self.animation_running:
            speed = random.uniform(2, 5) * self.speed_multiplier
            angle = random.uniform(0, 2 * math.pi)
            velocity_x = speed * math.cos(angle)
            velocity_y = speed * math.sin(angle)
        
        # Oval top çiz
        ball_id = self.canvas.create_oval(
            x - radius_x, y - radius_y,
            x + radius_x, y + radius_y,
            fill=color,
            outline="black",
            width=2
        )
        
        # Top bilgilerini kaydet
        ball = {
            'id': ball_id,
            'x': x,
            'y': y,
            'size': size,
            'width': width,
            'height': height,
            'radius_x': radius_x,
            'radius_y': radius_y,
            'color': color,
            'velocity_x': velocity_x,
            'velocity_y': velocity_y
        }
        
        self.balls.append(ball)
        print(f"Top eklendi: {len(self.balls)}. top, renk: {color}, boyut: {size} (oval)")
    
    def start_animation(self):
        """Animasyonu başlat"""
        print(f"START butonuna tıklandı. Top sayısı: {len(self.balls)}")
        
        if not self.animation_running:
            self.animation_running = True
            # Tüm toplara rastgele hız ver (duruyorlarsa)
            for ball in self.balls:
                if ball['velocity_x'] == 0 and ball['velocity_y'] == 0:
                    speed = random.uniform(2, 5) * self.speed_multiplier
                    angle = random.uniform(0, 2 * math.pi)
                    ball['velocity_x'] = speed * math.cos(angle)
                    ball['velocity_y'] = speed * math.sin(angle)
                    print(f"Top hızlandı: vx={ball['velocity_x']:.2f}, vy={ball['velocity_y']:.2f}")
            
            # Animasyon döngüsünü başlat
            print("Animasyon başlatılıyor...")
            self.animate()
        else:
            print("Animasyon zaten çalışıyor")
    
    def stop_animation(self):
        """Animasyonu durdur"""
        self.animation_running = False
    
    def reset_animation(self):
        """Ekranı sıfırla"""
        self.animation_running = False
        # Tüm topları sil
        for ball in self.balls:
            self.canvas.delete(ball['id'])
        self.balls.clear()
        self.speed_multiplier = 1.0
    
    def speed_up(self):
        """Hızı artır"""
        if self.speed_multiplier > 0:
            old_multiplier = self.speed_multiplier
            self.speed_multiplier += 0.5
            # Hareket halindeki topların hızını artır
            for ball in self.balls:
                if ball['velocity_x'] != 0 or ball['velocity_y'] != 0:
                    ratio = self.speed_multiplier / old_multiplier
                    ball['velocity_x'] *= ratio
                    ball['velocity_y'] *= ratio
    
    def animate(self):
        """Animasyon döngüsü - toplar hareket eder ve kenarlara değince sekeler"""
        if not self.animation_running:
            return
        
        # Toplar varsa hareket ettir
        if len(self.balls) == 0:
            # Top yoksa döngüyü durdur
            self.animation_running = False
            return
        
        for ball in self.balls:
            # Yeni pozisyon hesapla
            new_x = ball['x'] + ball['velocity_x']
            new_y = ball['y'] + ball['velocity_y']
            
            radius_x = ball['radius_x']
            radius_y = ball['radius_y']
            
            # Sol/sağ kenar kontrolü - sekme (hızı tersine çevir)
            if new_x - radius_x <= 0:
                ball['velocity_x'] = -ball['velocity_x']  # Yönü tersine çevir (sekme)
                new_x = radius_x
            elif new_x + radius_x >= self.canvas_width:
                ball['velocity_x'] = -ball['velocity_x']  # Yönü tersine çevir (sekme)
                new_x = self.canvas_width - radius_x
            
            # Üst/alt kenar kontrolü - sekme (hızı tersine çevir)
            if new_y - radius_y <= 0:
                ball['velocity_y'] = -ball['velocity_y']  # Yönü tersine çevir (sekme)
                new_y = radius_y
            elif new_y + radius_y >= self.canvas_height:
                ball['velocity_y'] = -ball['velocity_y']  # Yönü tersine çevir (sekme)
                new_y = self.canvas_height - radius_y
            
            # Pozisyonu güncelle
            ball['x'] = new_x
            ball['y'] = new_y
            
            # Canvas'da oval topu hareket ettir
            self.canvas.coords(
                ball['id'],
                new_x - radius_x, new_y - radius_y,
                new_x + radius_x, new_y + radius_y
            )
        
        # Canvas'ı güncelle
        self.canvas.update_idletasks()
        
        # Bir sonraki frame
        self.root.after(16, self.animate)  # ~60 FPS


def main():
    root = tk.Tk()
    app = BallAnimation(root)
    root.mainloop()


if __name__ == "__main__":
    main()
