# 簡易會員系統(註冊、登入、登出、改密碼)


# database connect

import pymongo
client = pymongo.MongoClient("mongodb+srv://帳號:密碼@mycluster.6wtnc1a.mongodb.net/?retryWrites=true&w=majority")
db=client.my_member_system

print("資料庫練線成功")

import json

# 初始化 Flask伺服器
from flask import *
app=Flask(
    __name__,static_folder="static",static_url_path="/" )

app.secret_key="any string but secret!"  

# 處理路由

@app.route("/")
def index():
    lang=request.headers.get("accept-language")
    if lang.startswith("en"):
        return redirect("/en/")
        
    else:
        return redirect("/zh/")
    
@app.route("/en/")
def index_en():
    return render_template("index_en.html")

@app.route("/zh/")
def index_zh():
    return render_template("index.html")
    # return render_template('index.html')

@app.route("/member")
def member():
    if "nickname" in session:
        return render_template("member.html")
    else:
        return redirect("/")

@app.route("/error")
def error():
    message=request.args.get("msg","發生錯誤，請聯繫客服")
    return render_template("error.html",message=message)

@app.route("/signup", methods=["POST"])
def signup():
    nickname=request.form["nickname"]
    gender=request.form["gender"]
    birthday=request.form["birthday"]
    email=request.form["email"]
    password=request.form["password"]

    collection=db.user
    result=collection.find_one({"email":email})

    if result != None:
        return redirect("/error?msg=信箱已經被註冊了/The mail had account already.")
    collection.insert_one({
        "nickname": nickname,
        "gender": gender,
        "birthday" : birthday,
        "email": email,
        "password":password

    })
    return redirect("/")

@app.route("/signin", methods=["POST"])
def signin():
    email=request.form['email']
    password=request.form['password']

    collection=db.user
    result=collection.find_one({"$and":[{"email":email},{"password":password}]})

    if result == None:
        return redirect("/error?msg=您尚未註冊信箱或密碼錯誤")
    session['nickname']=result['nickname']
    return redirect("/member")

@app.route("/signout")
def signout():
    del session['nickname']
    return redirect('/')

@app.route("/changemail", methods=["POST"])
def changemail():   
    return render_template('mail.html')

@app.route("/checkmail", methods=["POST"])
def checkmail():
    email=request.form['email']

    collection=db.user
    result=collection.find_one({'email':email})

    if result == None:
        return redirect("/error?msg=您尚未註冊信箱或信箱輸入錯誤")
    session['email']=result['email']
    return render_template("password.html")

@app.route("/forget", methods=["POST"])
def forget():
    
    password0=request.form['password0']
    password1=request.form['password1']

    collection=db.user

    if password0 != password1:
        return redirect("/error?msg=密碼不一致，請重新輸入")
    # session['password1']=password1
    

    update_result=collection.update_one({"email":session['email']},{'$set':{'password':password1}})
    print('密碼更新成功，請重新登入',update_result.matched_count)
    
    return redirect("/")
        




app.run(debug=True,port=5500)
