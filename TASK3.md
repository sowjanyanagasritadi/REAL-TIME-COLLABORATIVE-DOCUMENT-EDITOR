from flask import Flask, render_template, request, jsonify
from flask_socketio import SocketIO, emit
from pymongo import MongoClient

app = Flask(_name_)
app.config['SECRET_KEY'] = 'secret!'
socketio = SocketIO(app, cors_allowed_origins='*')

# MongoDB setup
client = MongoClient("mongodb://localhost:27017/")
db = client['editor']
doc_collection = db['documents']

@app.route('/')
def index():
    return render_template('editor.html')

@app.route('/load')
def load_document():
    doc = doc_collection.find_one({"name": "default"})
    if doc:
        return jsonify({"content": doc["content"]})
    return jsonify({"content": ""})

@socketio.on('edit')
def handle_edit(data):
    content = data['content']
    doc_collection.update_one({"name": "default"}, {"$set": {"content": content}}, upsert=True)
    emit('update', {'content': content}, broadcast=True)

if _name_ == '_main_':
    socketio.run(app, debug=True)
