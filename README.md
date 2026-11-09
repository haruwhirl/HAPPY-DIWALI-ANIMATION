import math
import random
import tkinter as tk

try:
  import winsound
except ImportError:
  winsound = None


class DiwaliRocketApp:

  def __init__(self, root):
    self.root = root
    self.root.title("Happy Diwali Rocket Animation")
    self.root.geometry("800x650")
    self.root.config(bg="#050515")
    self.root.resizable(False, False)

    # Canvas for animations
    self.canvas = tk.Canvas(
        root, width=800, height=530, bg="#050515", highlightthickness=0
    )
    self.canvas.pack(pady=10)

    # Draw twinkling background stars
    self.stars = []
    for _ in range(60):
      x = random.randint(0, 800)
      y = random.randint(0, 530)
      size = random.randint(1, 3)
      star = self.canvas.create_oval(    x, y, x + size, y + size, fill="white", outline=""
      )
      self.stars.append(star)

    # Control Panel Frame
    control_frame = tk.Frame(root, bg="#050515")
    control_frame.pack(fill="x", pady=10)

    # Start Button
    self.start_btn = tk.Button(
        control_frame,
        text="🚀 Start Diwali Celebration",
        font=("Arial", 14, "bold"),
        bg="#FF9933",
        fg="white",
        activebackground="#FF6600",
        activeforeground="white",
        padx=15,
        pady=5,
        command=self.start_sequence,
    )
    self.start_btn.pack()

    # Status / Instruction Label
    self.status_lbl = tk.Label(
        root,
        text=(
            "Click 'Start Diwali Celebration', then click on the rocket to"
            " launch it!"
        ),
        font=("Arial", 11, "italic"),
        bg="#050515",
        fg="#A0A0C0",
    )
    self.status_lbl.pack(pady=5)

    # Animation state variables
    self.rocket_id = None
    self.rocket_flame = None
    self.rx = 400
    self.ry = 420
    self.is_running = False
    self.exploded = False
    self.particles = []

  def play_sound(self, sound_type):
    """Plays cross-platform sound effects or fallback beeps."""
    if winsound:
      try:
        if sound_type == "launch":
          winsound.Beep(1200, 150)
        elif sound_type == "burst":
          winsound.Beep(2500, 400)
      except RuntimeError:
        pass

  def start_sequence(self):
    if self.is_running:
      return
    self.is_running = True
    self.exploded = False
    self.start_btn.config(state="disabled")

    # Clear any previous particles or text
    self.canvas.delete("burst_element")
    if self.rocket_id:
      self.canvas.delete(self.rocket_id)
    if self.rocket_flame:
      self.canvas.delete(self.rocket_flame)

    # Reset rocket position at the launchpad
    self.rx = 400
    self.ry = 420

    # Draw Rocket Body (Main fuselage)
    self.rocket_id = self.canvas.create_rectangle(
        self.rx - 8,
        self.ry - 25,
        self.rx + 8,
        self.ry + 20,
        fill="#FF3333",
        outline="yellow",
        width=2,
    )
    # Rocket Nose Cone
    self.nose_id = self.canvas.create_polygon(
        self.rx - 8,
        self.ry - 25,
        self.rx + 8,
        self.ry - 25,
        self.rx,
        self.ry - 42,
        fill="#FFCC00",
        outline="",
    )
    # Left Fin
    self.fin_l = self.canvas.create_polygon(
        self.rx - 8,
        self.ry + 10,
        self.rx - 18,
        self.ry + 25,
        self.rx - 8,
        self.ry + 20,
        fill="#FF9900",
        outline="",
    )
    # Right Fin
    self.fin_r = self.canvas.create_polygon(
        self.rx + 8,
        self.ry + 10,
        self.rx + 18,
        self.ry + 25,
        self.rx + 8,
        self.ry + 20,
        fill="#FF9900",
        outline="",
    )
    # Flame stub
    self.rocket_flame = self.canvas.create_polygon(
        self.rx - 5,
        self.ry + 20,
        self.rx + 5,
        self.ry + 20,
        self.rx,
        self.ry + 32,
        fill="#FFCC00",
        outline="",
    )

    # Group all rocket parts into a single tag for easy clicking/moving
    self.canvas.addtag_overlapping(
        "rocket_group",
        self.rx - 20,
        self.ry - 45,
        self.rx + 20,
        self.ry + 35,
    )

    # Bind click event specifically to the rocket group
    self.canvas.tag_bind(
        "rocket_group", "<Button-1>", lambda e: self.launch_rocket()
    )

    self.status_lbl.config(
        text="🚀 Rocket ready on launchpad! Click the rocket to launch it upwards!"
    )

  def launch_rocket(self):
    if not self.is_running or self.exploded:
      return

    # Unbind click once launched
    self.canvas.tag_unbind("rocket_group", "<Button-1>")
    self.status_lbl.config(text="🔥 Rocket launched! Heading into the sky...")
    self.play_sound("launch")

    # Animate rocket upward flight
    self.animate_flight()

  def animate_flight(self):
    if self.ry > 130:
      # Move all rocket parts upward
      self.canvas.move(self.rocket_id, 0, -8)
      self.canvas.move(self.nose_id, 0, -8)
      self.canvas.move(self.fin_l, 0, -8)
      self.canvas.move(self.fin_r, 0, -8)
      self.canvas.move(self.rocket_flame, 0, -8)
      self.ry -= 8

      # Flicker flame effect
      flame_color = random.choice(["#FFCC00", "#FF3300", "#FFFFFF"])
      self.canvas.itemconfig(self.rocket_flame, fill=flame_color)

      self.root.after(20, self.animate_flight)
    else:
      # Reach explosion point
      self.trigger_burst()

  def trigger_burst(self):
    self.exploded = True
    # Delete rocket elements
    for item in [
        self.rocket_id,
        self.nose_id,
        self.fin_l,
        self.fin_r,
        self.rocket_flame,
    ]:
      self.canvas.delete(item)

    self.play_sound("burst")

    # Display "Happy Diwali" greeting text
    self.canvas.create_text(
        400,
        140,
        text="✨ HAPPY DIWALI ✨",
        font=("Arial", 36, "bold"),
        fill="#FFD700",
        tags="burst_element",
    )
    self.canvas.create_text(
        400,
        190,
        text="May your life be filled with light, joy, and prosperity!",
        font=("Arial", 16, "italic"),
        fill="#00FFFF",
        tags="burst_element",
    )

    # Generate colorful firework spark particles
    self.particles = []
    colors = ["#FF0055", "#00FF66", "#00CCFF", "#FFFF00", "#FF9900", "#FF66FF"]
    for _ in range(70):
      angle = random.uniform(0, 2 * math.pi)
      speed = random.uniform(2, 9)
      vx = speed * math.cos(angle)
      vy = speed * math.sin(angle)
      color = random.choice(colors)
      size = random.randint(3, 6)
      p = self.canvas.create_oval(
          400, 130, 400 + size, 130 + size, fill=color, outline="", tags="burst_element"
      )
      self.particles.append({"id": p, "vx": vx, "vy": vy, "life": 60})

    self.animate_burst_particles()

  def animate_burst_particles(self):
    active_particles = False
    for p in self.particles:
      if p["life"] > 0:
        active_particles = True
        self.canvas.move(p["id"], p["vx"], p["vy"])
        # Add slight gravity effect p["vy"] += 0.15
        p["life"] -= 1

    if active_particles:
      self.root.after(30, self.animate_burst_particles)
    else:
      self.status_lbl.config(
          text="✨ Celebration complete! Click 'Start Diwali Celebration' to"
          " replay."
      )
      self.start_btn.config(state="normal")
      self.is_running = False


if __name__ == "__main__":
  root = tk.Tk()
  app = DiwaliRocketApp(root)
  root.mainloop()
