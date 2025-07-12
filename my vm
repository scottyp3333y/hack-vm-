import sys, os, subprocess, shutil, hashlib, json
from PIL import Image, ImageFilter

USE_GUI = True
try:
    from PyQt5.QtWidgets import (
        QApplication, QWidget, QVBoxLayout, QHBoxLayout, QLabel, QPushButton,
        QFileDialog, QLineEdit, QMessageBox, QCheckBox, QFrame, QGroupBox
    )
    from PyQt5.QtGui import QFont, QPixmap
except ModuleNotFoundError:
    USE_GUI = False

class QLearningAgent:
    def __init__(self):
        self.q_table = {}
        self.alpha = 0.5
        self.gamma = 0.9

    def update(self, state, action, reward, next_state):
        self.q_table.setdefault(state, {})
        self.q_table[state][action] = (
            (1 - self.alpha) * self.q_table[state].get(action, 0) +
            self.alpha * (reward + self.gamma * max(self.q_table.get(next_state, {}).values(), default=0))
        )

    def choose_action(self, state):
        actions = self.q_table.get(state, {})
        if not actions:
            return None
        return max(actions, key=actions.get)

def cli_launcher():
    print("=== CLI HackVM Launcher ===")
    try:
        iso = input("Enter path to ISO or IMG: ")
    except OSError:
        print("Input not available. Using fallback ISO path.")
        iso = "fallback.iso"

    if not os.path.exists(iso):
        print(f"Error: file not found -> {iso}")
        return

    try:
        ram = input("Memory (MB) [2048]: ") or "2048"
    except OSError:
        print("Input not available. Defaulting to 2048 MB RAM.")
        ram = "2048"

    try:
        disk = input("Optional disk save size (GB, blank=none): ")
    except OSError:
        print("Input not available. No persistent disk will be created.")
        disk = ""

    args = ["qemu-system-x86_64", "-m", ram, "-enable-kvm"]
    if disk:
        disk_file = f"cli_hackvm_{hashlib.sha1(iso.encode()).hexdigest()[:8]}.qcow2"
        if not os.path.exists(disk_file):
            subprocess.run(["qemu-img", "create", "-f", "qcow2", disk_file, f"{disk}G"], check=True)
        args += ["-drive", f"file={disk_file},format=qcow2"]

    if iso.lower().endswith('.iso'):
        args += ["-cdrom", iso, "-boot", "d"]
    else:
        args += ["-hda", iso]

    print("Launching QEMU:", ' '.join(args))
    try:
        subprocess.run(args)
    except Exception as e:
        print(f"Failed to launch VM: {e}")

if __name__ == '__main__':
    if not USE_GUI:
        cli_launcher()
    else:
        class SmartHackVM(QWidget):
            def __init__(self):
                super().__init__()
                self.setWindowTitle("HackVM V3 – Pro AI-Enhanced Loader")
                self.setGeometry(50, 50, 1200, 700)
                self.ai_agent = QLearningAgent()
                self.last_state = None
                self.last_action = None
                self.img_path = None
                self.result_img = None
                self.selected_file = None
                self.disk_file = None
                main_layout = QHBoxLayout()
                main_layout.addLayout(self.build_image_panel())
                main_layout.addWidget(self._vertical_separator())
                main_layout.addLayout(self.build_vm_panel())
                self.setLayout(main_layout)

            def _vertical_separator(self):
                sep = QFrame()
                sep.setFrameShape(QFrame.VLine)
                sep.setLineWidth(2)
                return sep

            def build_image_panel(self):
                layout = QVBoxLayout()
                title = QLabel("📷 AI Image Processor")
                title.setFont(QFont("Arial", 16))
                layout.addWidget(title)
                self.preview = QLabel("No image loaded")
                self.preview.setFixedSize(350, 350)
                self.preview.setFrameStyle(QFrame.Box | QFrame.Raised)
                layout.addWidget(self.preview)
                btns = QHBoxLayout()
                load_btn = QPushButton("Load Image")
                load_btn.clicked.connect(self.load_image)
                btns.addWidget(load_btn)
                proc_btn = QPushButton("Run AI Process")
                proc_btn.clicked.connect(self.run_ai)
                btns.addWidget(proc_btn)
                layout.addLayout(btns)
                self.save_btn = QPushButton("Download Result")
                self.save_btn.setEnabled(False)
                self.save_btn.clicked.connect(self.save_image)
                layout.addWidget(self.save_btn)
                return layout

            def load_image(self):
                path, _ = QFileDialog.getOpenFileName(self, "Select Image", "", "Images (*.png *.jpg *.bmp)")
                if not path:
                    return
                self.img_path = path
                pix = QPixmap(path).scaled(350, 350, aspectRatioMode=1)
                self.preview.setPixmap(pix)
                self.save_btn.setEnabled(False)

            def run_ai(self):
                if not self.img_path:
                    QMessageBox.warning(self, "No Image", "Please load an image first!")
                    return
                try:
                    img = Image.open(self.img_path).convert("L")
                    processed = img.filter(ImageFilter.FIND_EDGES)
                    tmp = os.path.join(os.getcwd(), "ai_result.png")
                    processed.save(tmp)
                    self.result_img = tmp
                    pix = QPixmap(tmp).scaled(350, 350, aspectRatioMode=1)
                    self.preview.setPixmap(pix)
                    self.save_btn.setEnabled(True)
                    QMessageBox.information(self, "AI Done", "Image processed successfully.")
                except Exception as e:
                    QMessageBox.critical(self, "Processing Error", str(e))

            def save_image(self):
                if not self.result_img:
                    return
                dst, _ = QFileDialog.getSaveFileName(self, "Save Result As", os.path.basename(self.result_img))
                if dst:
                    try:
                        shutil.copy(self.result_img, dst)
                        QMessageBox.information(self, "Downloaded", f"Saved to {dst}")
                    except Exception as e:
                        QMessageBox.critical(self, "Save Error", str(e))

            def build_vm_panel(self):
                layout = QVBoxLayout()
                title = QLabel("🛡️ HackVM V3 – Pro Loader")
                title.setFont(QFont("Arial", 16))
                layout.addWidget(title)
                self.file_label = QLabel("No disk image selected.")
                select_btn = QPushButton("Select ISO/IMG File")
                select_btn.clicked.connect(self.select_iso)
                layout.addWidget(select_btn)
                layout.addWidget(self.file_label)
                mem_group = QGroupBox("Memory Settings (MB)")
                mem_layout = QHBoxLayout()
                self.ram_input = QLineEdit("4096")
                mem_layout.addWidget(self.ram_input)
                mem_group.setLayout(mem_layout)
                layout.addWidget(mem_group)
                disk_group = QGroupBox("Disk Save (GB)")
                disk_layout = QHBoxLayout()
                self.disk_input = QLineEdit("")
                disk_layout.addWidget(self.disk_input)
                disk_group.setLayout(disk_layout)
                layout.addWidget(disk_group)
                self.boot_checkbox = QCheckBox("Boot from disk after installer exits")
                layout.addWidget(self.boot_checkbox)
                suggest_btn = QPushButton("💡 Smart Suggest")
                suggest_btn.clicked.connect(self.suggest_config)
                layout.addWidget(suggest_btn)
                install_ai_btn = QPushButton("Install Q-Learning AI Agent")
                install_ai_btn.clicked.connect(self.install_ai_agent)
                layout.addWidget(install_ai_btn)
                launch_btn = QPushButton("🚀 Launch Virtual Machine")
                launch_btn.clicked.connect(self.launch_vm)
                layout.addWidget(launch_btn)
                self.status = QLabel("Status: Idle")
                layout.addWidget(self.status)
                return layout

            def select_iso(self):
                path, _ = QFileDialog.getOpenFileName(self, "Open Disk Image", "", "Disk Images (*.iso *.img)")
                if path:
                    self.selected_file = path
                    self.file_label.setText(f"Selected: {os.path.basename(path)}")
                    self.disk_file = f"hackvm_{hashlib.sha1(path.encode()).hexdigest()[:8]}.qcow2"
                else:
                    self.file_label.setText("No disk image selected.")

            def install_ai_agent(self):
                QMessageBox.information(self, "AI Agent", "Q-Learning AI Agent installed.")
                self.status.setText("AI Agent installed.")

            def suggest_config(self):
                if not self.selected_file:
                    QMessageBox.warning(self, "Missing File", "Select a file before suggesting.")
                    return
                state = f"file={os.path.basename(self.selected_file)},disk={bool(self.disk_input.text())}"
                best = self.ai_agent.choose_action(state)
                if best and best.startswith("ram="):
                    self.ram_input.setText(best.split("=")[1])
                    QMessageBox.information(self, "AI Suggestion", f"Best RAM config: {best}")
                else:
                    QMessageBox.information(self, "AI Suggestion", "Not enough data yet.")

            def launch_vm(self):
                if not getattr(self, 'selected_file', None):
                    QMessageBox.warning(self, "No File", "Please select an ISO or IMG file first.")
                    return
                try:
                    ram = int(self.ram_input.text())
                except ValueError:
                    QMessageBox.warning(self, "Invalid RAM", "Enter a numeric RAM value.")
                    return
                disk_size = self.disk_input.text().strip()
                args = ["qemu-system-x86_64", "-m", str(ram), "-enable-kvm", "-display", "gtk"]
                if disk_size:
                    if not os.path.exists(self.disk_file):
                        subprocess.run(["qemu-img", "create", "-f", "qcow2", self.disk_file, f"{disk_size}G"], check=True)
                    args += ["-drive", f"file={self.disk_file},format=qcow2"]
                if self.selected_file.lower().endswith('.iso'):
                    args += ["-cdrom", self.selected_file]
                    boot_opt = "once=d" if self.boot_checkbox.isChecked() and disk_size else "d"
                    args += ["-boot", boot_opt]
                else:
                    args += ["-hda", self.selected_file]

                current_state = f"file={os.path.basename(self.selected_file)},disk={bool(disk_size)}"
                chosen_action = f"ram={ram}"
                reward = 10 if disk_size else 1
                if self.last_state:
                    self.ai_agent.update(self.last_state, self.last_action, reward, current_state)
                self.last_state = current_state
                self.last_action = chosen_action

                self.status.setText("Launching VM...")
                try:
                    subprocess.run(args, check=True)
                    self.status.setText("VM exited normally.")
                except Exception as e:
                    self.status.setText(f"Error: {e}")

        app = QApplication(sys.argv)
        window = SmartHackVM()
        window.show()
        app.exec_()
