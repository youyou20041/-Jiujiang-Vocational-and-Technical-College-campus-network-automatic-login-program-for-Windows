import win32com.client
import tkinter as tk
import winshell
import requests
import shutil
import json
import sys
import os
from tkinter import ttk  

#登录校园网模块------------------
def save_credentials(credentials):
    with open('credentials.json', 'w') as file:
        json.dump(credentials, file)

def load_credentials():
    if os.path.exists('credentials.json'):
        with open('credentials.json', 'r') as file:
            return json.load(file)
    return None

def update_credentials_entry(credentials):
    if credentials:
        entry_username.delete(0, tk.END)
        entry_username.insert(0, credentials['username'])
        entry_password.delete(0, tk.END)
        entry_password.insert(0, credentials['password'])
        option_var.set(credentials['provider'])
        auto_login_var.set(credentials.get('auto_login', 0))
        '''
def auto_login(provider, username, password, on_success, on_failure):

    #login_url = 'http://10.31.0.10:801/eportal/portal/login'
    user_account_value = option_var.get()
    params = {
        'callback': 'dr1003',
        #'login_method': '1',
        'user_account': f'{username}@{providers[user_account_value]}',
        'user_password': password,
        ## ... 添加其他所有必要的参数 ...
        #'jsVersion': '4.1.3',
        #'terminal_type': '1',
        #'lang': 'zh-cn',
        
    }
'''
    '''
    try:
        response = requests.get(login_url, params=params)
        #登录成功 #dr1003({"result":1,"msg":"Portal协议认证成功！"});
        if '"result":1' in response.text and '"Portal协议认证成功！"' in response.text:
            save_credentials({'provider': provider, 'username': username, 'password': password, 'auto_login': auto_login_var.get()})
            on_success()
        #其他电脑已登陆 #dr1003({"result": 0,"msg": "Rad:109026004|109020109|Reject by concurrency control.","ret_code": 1});
        elif '"result": 0' in response.text and '"Rad:109026004|109020109|Reject by concurrency control."' in response.text and '"ret_code": 1' in response.text:
            label_message.config(text="其他电脑已登陆", fg="red")
        #账号不存在/选错运营商 #dr1003({"result":0,"msg":"账号不存在","ret_code":1});
        elif '"result":0' in response.text and '"账号不存在"' in response.text and '"ret_code":1' in response.text:
            label_message.config(text="账号不存在/选错运营商", fg="red")
        #密码错误 #dr1003({"result":0,"msg":"密码错误","ret_code":1});
        elif '"result":0' in response.text and '"密码错误"' in response.text and '"ret_code":1' in response.text:
             label_message.config(text="密码错误", fg="red")
        # 重复登录
        else:
            label_message.config(text="已登上了,重复登录", fg="red")

    except Exception as e:
        print(f"请求发生错误: {e}")
        on_failure()
'''
def on_login_click():
    provider = option_var.get()
    username = entry_username.get()
    password = entry_password.get()
    auto_login(provider, username, password, on_success, on_failure)

def on_success():
    label_message.config(text="登录成功", fg="green")
    root.after(800,root.destroy)

def on_failure():
    label_message.config(text="登陆失败", fg="red")
#-------------------------------------------------------------

def create_shortcut(target, shortcut_name, start_in="", icon=""):
    """"
    #创建快捷方式
    :param target: 快捷方式指向的目标路径
    :param shortcut_name: 快捷方式的名称
    :param start_in: 快捷方式的起始位置（工作目录）
    :param icon: 快捷方式的图标路径
    """
    #desktop = winshell.desktop()  
    path = os.path.join(start_in, shortcut_name + ".lnk")  
    shell = win32com.client.Dispatch("WScript.Shell")  
    shortcut = shell.CreateShortcut(path)  
    shortcut.Targetpath = target  
    shortcut.WorkingDirectory = start_in  
    if icon:
        shortcut.IconLocation = icon  
    shortcut.save()  

def add_to_startup(shortcut_name , exe_path):
    """
    将快捷方式添加到开机自启动
    :param shortcut_name: 快捷方式的名称
    """

    start_in=os.path.dirname(exe_path)
    create_shortcut(exe_path, "MyAppShortcut", start_in)
    startup_dir = winshell.startup() 
    startup_shortcut = os.path.join(startup_dir, shortcut_name + ".lnk") 
    shutil.copyfile(os.path.join(start_in, shortcut_name + ".lnk"), startup_shortcut) 


def is_running_from_exe():
    return getattr(sys, 'frozen', False)
#-------------------------------------------------------------

executable_path = sys.executable if is_running_from_exe() else os.path.abspath(__file__)

first_run_flag = os.path.join(os.path.dirname(executable_path), 'first_run.flag')
if not os.path.exists(first_run_flag):
    with open(first_run_flag, 'w') as f:
        f.write('This is a flag file indicating the app has been run at least once.')

    # 添加到开机自启动
    add_to_startup("MyAppShortcut", executable_path)

popup_window = None

def show_popup():
    global popup_window

    # 如果已经存在弹出窗口，则执行抖动效果
    if popup_window and popup_window.winfo_exists():
        shake_window(popup_window)
        popup_window.attributes('-topmost', 1)
        return

    popup_window = tk.Toplevel(root)
    popup_window.title("关于船电2303的一位同学")

    # Calculate the center position for the popup window
    screen_width = root.winfo_screenwidth()
    screen_height = root.winfo_screenheight()
    popup_width = 380
    popup_height = 230
    x = int((screen_width - popup_width) / 3)
    y = int((screen_height - popup_height) / 3)
    popup_window.geometry(f"{popup_width}x{popup_height}+{x}+{y}")

    message_text = "本程序完全免费没有用于任何盈利项目,请切勿上当受骗!\n重复登录的意思是你现在已经登上了校园网,先注销登录在使用\n账号是学号,密码是身份证后六位\n(以后就不知道了哦,如果错误问问营业厅老板)\n遇到问题?请联系作者\nQQ群:765466404(欢迎大家加群聊天或者问问题qvq)\n微信:linxiaoyi0xy\n希望能找到志同道合的同学一起探索计算机的世界\n(想参加比赛但是找不到人一起啊啊啊啊)\nby\nhwq和另一位不愿透露姓名的同学\n更新网盘:https://wwph.lanzout.com/b048y930j\n密码:fb4z"
    message_label = tk.Text(popup_window, wrap="word", height=300, width=300)
    message_label.insert(tk.END, message_text)
    message_label.tag_configure("red_tag", foreground="red")
    message_label.tag_add("red_tag", "1.0", "1.26","6.4","6.13","7.3","7.15")
    message_label.config(state=tk.DISABLED)
    message_label.pack(padx=10, pady=20)

def shake_window(window):
    x, y = window.winfo_x(), window.winfo_y()

    for _ in range(5):
        x += 10
        window.geometry(f"+{x}+{y}")
        window.update_idletasks()
        window.after(50)

        x -= 10
        window.geometry(f"+{x}+{y}")
        window.update_idletasks()
        window.after(50)

root = tk.Tk()
root.title("免费校园网自动登录")

window_width = 300
window_height = 200
screen_width = root.winfo_screenwidth()
screen_height = root.winfo_screenheight()
x = int((screen_width / 2) - (window_width / 2))
y = int((screen_height / 2) - (window_height / 2))
root.geometry(f'{window_width}x{window_height}+{x}+{y}')

tk.Label(root, text="账号").pack()
entry_username = tk.Entry(root)
entry_username.pack()

tk.Label(root, text="密码").pack()
entry_password = tk.Entry(root, show="*")
entry_password.pack()


frame = tk.Frame(root)
frame.pack()


option_var = tk.StringVar(value="telecom") 
providers = {
    "电信": "telecom",
    "联通": "unicom",
    "移动": "cmcc"
}

for provider, value in providers.items():
    rb = ttk.Radiobutton(frame, text=provider, variable=option_var, value=provider)
    rb.pack(side=tk.LEFT)
#option_var = ttk.Radiobutton(root)  
#option_var['values'] = providers  
#option_var.current(0)  # 设置默认选项为第一个选项  
#option_var.grid(column=0, row=0)  
#option_var.pack()

auto_login_var = tk.IntVar(value=0)  # 默认不勾选
check_auto_login = tk.Checkbutton(root, text="自动登录", variable=auto_login_var)
check_auto_login.pack()

#option_menu = tk.OptionMenu(root, option_var, *providers.keys())
#option_menu.pack()

button_show_popup = tk.Button(root, text="技术支持点我", command=show_popup)
button_show_popup.pack(side="left", padx=10)


button_login = tk.Button(root, text="登录", command=on_login_click)
button_login.pack(side="left",padx=30)


label_message = tk.Label(root, text="")
label_message.pack(side="right",padx=1)

credentials = load_credentials()
if credentials:
    update_credentials_entry(credentials)

if credentials and credentials.get('auto_login'):
    on_login_click()


root.resizable(width=False, height=False)
root.attributes('-topmost', 1)

root.mainloop()
#pyinstaller --onefile --windowed creatfile.py
