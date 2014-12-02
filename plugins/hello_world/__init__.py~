import os
import datetime
from flask import Flask, render_template, Blueprint, send_from_directory,\
    render_template_string, current_app, request, redirect, url_for
from flask.ext.plugins import connect_event, emit_event, Plugin
import psycopg2
import time
from app import AppPlugin, app
from werkzeug import secure_filename

__plugin__ = "HelloWorld"


def getalbum():
    conn = psycopg2.connect(database="albumdb",
                            user="postgres", password="postgres",
                            host="127.0.0.1", port="5432")
    cur = conn.cursor()
    cur.execute('SELECT * FROM album ORDER BY "id" DESC')
    albums = cur.fetchall()        # all rows in table
    conn.commit()
    cur.close()
    conn.close()
    return albums


def getphoto(Photo_ID):
    conn = psycopg2.connect(database="albumdb",
                            user="postgres", password="postgres",
                            host="127.0.0.1", port="5432")
    cur = conn.cursor()
    cur.execute('SELECT * from photos where "pictureID" = %s', (Photo_ID,))
    photo = cur.fetchall()
    conn.commit()
    cur.close()
    conn.close()
    return photo


def getDesc(ID):
    conn = psycopg2.connect(database="albumdb",
                            user="postgres", password="postgres",
                            host="127.0.0.1", port="5432")
    cur = conn.cursor()
    cur.execute('SELECT "description" FROM album where "id" = %s', (ID,))
    desc = cur.fetchall()
    conn.commit()
    cur.close()
    conn.close()
    return desc


def getphotos(ID):
    conn = psycopg2.connect(database="albumdb",
                            user="postgres", password="postgres",
                            host="127.0.0.1", port="5432")
    cur = conn.cursor()
    cur.execute('SELECT * FROM photos where "albumid" = %s', (ID,))
    photos = cur.fetchall()
    conn.commit()
    cur.close()
    conn.close()
    return photos


def add_photo(albumID, name, phototype, today):
    conn = psycopg2.connect(database="albumdb",
                            user="postgres", password="postgres",
                            host="127.0.0.1", port="5432")
    cur = conn.cursor()
    cur.execute("INSERT INTO photos(name, creation, albumid, type)"
                "VALUES (%s, %s, %s, %s)", (name, today, albumID, phototype))
    cover = name + '_' + albumID + '.' + phototype
    cur.execute("UPDATE album set picture_number = picture_number + 1,"
                "cover_image = %s where id = %s", (cover, albumID))
    conn.commit()
    cur.close()
    conn.close()


def delete_photo_database(Photo_ID, albumID):
    conn = psycopg2.connect(database="albumdb",
                            user="postgres", password="postgres",
                            host="127.0.0.1", port="5432")
    cur = conn.cursor()
    cur.execute('SELECT * from photos where "pictureID" = %s', (Photo_ID,))
    filename = cur.fetchall()
    deletename = filename[0][0] + '_' + albumID + "." + filename[0][4]
    #os.remove(os.path.join(app.config['UPLOAD_FOLDER'], filename))
    cur.execute('DELETE from photos where "pictureID" = %s', (Photo_ID,))
    conn.commit()
    cur.execute("UPDATE album set picture_number = picture_number - 1"
                "where id = %s", (albumID,))
    conn.commit()
    cur.execute('SELECT * FROM photos where "albumid" = %s ORDER BY'
                '"pictureID" DESC', (albumID,))
    photos = cur.fetchall()
    if photos == []:
        cover = "default.png"
    else:
        cover = photos[0][0] + '_' + albumID + "." + photos[0][4]
    cur.execute("UPDATE album set cover_image = %s where id = %s",
                (cover, albumID))
    conn.commit()
    cur.close()
    conn.close()


def delete_album_database(albumID):
    conn = psycopg2.connect(database="albumdb",
                            user="postgres", password="postgres",
                            host="127.0.0.1", port="5432")
    cur = conn.cursor()
    photos = getphotos(albumID)
    for x in photos:
        delete_photo_database(x[2], albumID)
    cur.execute('DELETE from album where "id" = %s', (albumID,))
    conn.commit()
    cur.close()
    conn.close()


def update_album(ID):
    conn = psycopg2.connect(database="albumdb",
                            user="postgres", password="postgres",
                            host="127.0.0.1", port="5432")
    cur = conn.cursor()
    cur.execute('SELECT * FROM album where "id" = %s', ID)
    update = cur.fetchall()        # all rows in table
    conn.commit()
    cur.close()
    conn.close()
    return update


def show_album():
    return '<a href="http://127.0.0.1:5000/album">Photo Album</a>'


album = Blueprint("album", __name__, template_folder="templates")


@album.route("/")
def main():
    albums = getalbum()
    return render_template("album.html", albums=albums)


@album.route("/<ID>")
def show_photo(ID=None):
    photos = getphotos(ID)
    desc = getDesc(ID)
    return render_template("photo.html", photos=photos, ID=ID, desc=desc)


#show big photo
@album.route("/<ID>/<PhotoID>/show")
def show_big_photo(ID=None, PhotoID=None):
    Photo = getphoto(PhotoID)
    return render_template("show.html", Photo=Photo, ID=ID)


#delete photo
@album.route("/<ID>/<PhotoID>/delete")
def delete_photo(ID=None, PhotoID=None):
    photos = getphotos(ID)
    delete_photo_database(PhotoID, ID)
    s = "http://127.0.0.1:5000/album/" + ID
    return redirect(s)


#delete album
@album.route("/<ID>/delete")
def delete_album(ID=None, albumID=None):
    albums = getalbum()
    delete_album_database(ID)
    return redirect(url_for('album.main'))


#add photo code
@album.route("/<ID>/add")
def add(ID=None):
    return render_template("add.html", ID=ID)

app.config['UPLOAD_FOLDER'] = 'static/images/photos/'
app.config['ALLOWED_EXTENSIONS'] = set(['png', 'jpg', 'jpeg'])


def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1] in app.config['ALLOWED_EXTENSIONS']


@album.route('/<ID>/upload', methods=['POST'])
def upload(ID=None):
    file = request.files['file']
    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        today = datetime.date.today().strftime("%Y-%m-%d")
        albumID = ID
        name = filename.split('.')[0]
        phototype = filename.split('.')[1]
        file_ablum_name = name + '_' + albumID + '.' + phototype
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], file_ablum_name))
        add_photo(albumID, name, phototype, today)
        photos = getphotos(ID)
        s = "http://127.0.0.1:5000/album/" + ID
        return redirect(s)


#edit album code
@album.route("/<ID>/edit")
def edit(ID=None):
    update = update_album(ID)
    name = update[0][0]
    desc = update[0][2]
    return render_template("edit.html", name=name, desc=desc, ID=ID)


@album.route("/<ID>/edit2", methods=['POST'])
def welcome(ID=None):
    if request.method == 'POST':
        conn = psycopg2.connect(database="albumdb",
                                user="postgres", password="postgres",
                                host="127.0.0.1", port="5432")
        cur = conn.cursor()
        name = request.form.get('name', None)
        desc = request.form.get('desc', None)
        cover = request.form.get('cover', None)
        cur.execute('UPDATE album SET "name"=%s, "description"=%s'
                    'WHERE "id"=%s;', (name, desc, ID))
        conn.commit()
        cur.close()
        conn.close()
        s = "http://127.0.0.1:5000/album/" + ID
        return redirect(s)


#create album code
@album.route("/create")
def create():
    return render_template("create1.html")

app.config['UPLOAD_FOLDER'] = 'static/images/photos/'
app.config['ALLOWED_EXTENSIONS'] = set(['png', 'jpg', 'jpeg'])


def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1] in app.config['ALLOWED_EXTENSIONS']


@album.route('/create2', methods=['POST'])
def create2():
    name = request.form.get('albumName', None)
    desc = request.form.get('desc', None)
    default = "default.png"
    curtime = time.asctime(time.localtime(time.time()))
    conn = psycopg2.connect(database="albumdb",
                            user="postgres", password="postgres",
                            host="127.0.0.1", port="5432")
    cur = conn.cursor()
    cur.execute('INSERT INTO album ("name", "creation", "description",'
                '"picture_number", "cover_image") VALUES (%s, %s, %s, 0, %s)',
                (name, curtime, desc, default))
    conn.commit()
    cur.close()
    conn.close()
    return redirect(url_for('album.main'))


class HelloWorld(AppPlugin):
    def setup(self):
        self.register_blueprint(album, url_prefix="/album")
        connect_event("show_album", show_album)

    def install(self):
        # there is nothing to install
        pass

    def uninstall(self):
        # ... and nothing to uninstall
        pass
