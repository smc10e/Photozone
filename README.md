PhotoZone is a flask plugin that can be used on your personal website. It can provide a useful photo album plugin for you to upload pictures to the website. All you need to do is install this flask plugin. You can find the overall introduction of the project on the following website: http://ww2.cs.fsu.edu/~lin/photozone.html

To run the example website in your local computer. Please import the backup database "db" in postgres first(You need to create a new database called "albumdb"). And use command "python3.4 app.py" to run the website in the localserver. 

1.install postgres
sudo apt-get install postgresql

2.install psycopg2
sudo apt-get install python-psycopg2

3.install pgAdmin3
sudo apt-get install pgadmin3

4.set the password for postgress
  In a terminal, type:
  sudo -u postgres psql postgres

  Set a password for the "postgres" database role using the command:
  \password postgres

  In order to run the example, the password should be "postgres"

5.create database called "albumdb", you can either use the command line or GUI(pgadmin3)

6.import the backup database "db" to the database you just create, you can either use the command line or GUI(pgadmin3)

7.use command "python3.4 app.py" to run. Open your Browser and go to "http://127.0.0.1:5000/"

