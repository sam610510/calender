from PyQt6.QtWidgets import QApplication, QMainWindow, QLabel, QLineEdit, QPushButton, QTextEdit, QMessageBox, QFileDialog
from PyQt6.QtCore import Qt
import datetime


class CalendarApp(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("calender")
        self.resize(500, 500)
        #用setgeometry調按鈕和輸入、輸出框的大小位置
        #input box
        self.date_label = QLabel("Date(2023-05-01):", self)
        self.date_label.setGeometry(0, 0, 500, 30)
        self.date_edit = QLineEdit(self)
        self.date_edit.setGeometry(0, 30, 500, 30)

        self.importance_label = QLabel("importance_index(1 is most important, 10 is unimportant):", self)
        self.importance_label.setGeometry(0, 60, 500, 30)
        self.importance_edit = QLineEdit(self)
        self.importance_edit.setGeometry(0, 90, 500, 30)

        self.event_label = QLabel("event(作業繳交):", self)
        self.event_label.setGeometry(0, 120, 500, 30)
        self.event_edit = QLineEdit(self)
        self.event_edit.setGeometry(0,150, 500, 30)


        # button
        self.add_button = QPushButton("add", self)
        self.add_button.setGeometry(20, 200, 80, 30)
        self.add_button.clicked.connect(self.add_event)

        self.sort_button = QPushButton("sort", self)
        self.sort_button.setGeometry(210, 200, 80, 30)
        self.sort_button.clicked.connect(self.sort_events)

        self.load_button = QPushButton("load", self)
        self.load_button.setGeometry(400, 200, 80, 30)
        self.load_button.clicked.connect(self.load_file)
        

        # message
        self.text_edit = QTextEdit(self)
        self.text_edit.setGeometry(20, 250, 460, 230)
        self.text_edit.setReadOnly(True)

        # data
        self.events = []

    def add_event(self):
        date = self.date_edit.text()
        event = self.event_edit.text()
        importance = self.importance_edit.text()
        #判斷輸入的內容是否符合格式、是否格子內都有內容
        #用if、try、expect判斷，try的用法參考https://steam.oxxostudio.tw/category/python/basic/try-except.html
        if not date or not event or not importance:
            QMessageBox.warning(self, "false", "please input all data")
            return

        try:
            datetime.datetime.strptime(date, "%Y-%m-%d")
        except ValueError:
            QMessageBox.warning(self, "false", "invalid date")
            return

        try:
            importance = int(importance)
            if importance < 1 or importance > 10:
                raise ValueError
        except ValueError:
            QMessageBox.warning(self, "false", "importance_index need 1-10")
            return


        self.events.append((date, event, importance))

        # delete input box
        self.date_edit.clear()
        self.event_edit.clear()
        self.importance_edit.clear()
        self.display_events()
    #insertion sort from https://www.programiz.com/dsa/insertion-sort
    def sort_events(self):
        for i in range(1, len(self.events)):
            key = self.events[i]
            j = i - 1
            while j >= 0 and self.events[j][2] > key[2]:
                self.events[j + 1] = self.events[j]
                j -= 1
            self.events[j + 1] = key
        #datetime 排序 from https://www.nhooo.com/note/qahhqb.html
        self.events.sort(key=lambda x: datetime.datetime.strptime(x[0], "%Y-%m-%d"))

        self.display_events()
    
    def display_events(self):
        self.text_edit.clear()
        self.text_edit.append("Date/importance_index/event")
        for event in self.events:
            self.text_edit.append(f"{event[0]} / {event[2]} / {event[1]}")
#這部分花費了許多時間參考了許多方法和網站，其中https://stackoverflow.com/questions/59223603/qfiledialog-always-opens-behind-main-window 中的內容為主要使用方法
    def load_file(self):
        file_dialog = QFileDialog()
        file_dialog.setFileMode(QFileDialog.FileMode.ExistingFile)
        if file_dialog.exec() == QFileDialog.DialogCode.Accepted:
            file_path = file_dialog.selectedFiles()[0]
            self.parse_file(file_path)

    def parse_file(self, file_path):
        try:
            with open(file_path, "r") as file:
                self.events.clear()
                for line in file:
                    date, event, importance = line.strip().split(",")
                    self.events.append((date, event, int(importance)))
                self.sort_events()
        except Exception as e:
            QMessageBox.warning(self, "error", f"無法載入文件：{str(e)}")

    def keyPressEvent(self, event):
        if event.key() == Qt.Key.Key_Escape:
            self.close()


if __name__ == "__main__":
    app = QApplication([])
    window = CalendarApp()
    window.show()
    app.exec()
