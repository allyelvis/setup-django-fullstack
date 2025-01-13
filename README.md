Below is a Bash script that automates the setup of a Django backend integrated with a frontend (React) and a PostgreSQL database. The script includes virtual environment creation, dependency installation, project structure setup, and basic integration:

Script: setup_django_fullstack.sh

#!/bin/bash

# Script to set up Django backend, React frontend, and PostgreSQL database

# Set project name
PROJECT_NAME="myproject"
FRONTEND_DIR="frontend"
BACKEND_DIR="backend"

# Update and install dependencies
echo "Updating system and installing dependencies..."
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-venv python3-pip nodejs npm postgresql postgresql-contrib -y

# Set up PostgreSQL database
DB_NAME="mydb"
DB_USER="myuser"
DB_PASSWORD="mypassword"

echo "Setting up PostgreSQL database..."
sudo -u postgres psql -c "CREATE DATABASE $DB_NAME;"
sudo -u postgres psql -c "CREATE USER $DB_USER WITH PASSWORD '$DB_PASSWORD';"
sudo -u postgres psql -c "ALTER ROLE $DB_USER SET client_encoding TO 'utf8';"
sudo -u postgres psql -c "ALTER ROLE $DB_USER SET default_transaction_isolation TO 'read committed';"
sudo -u postgres psql -c "ALTER ROLE $DB_USER SET timezone TO 'UTC';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE $DB_NAME TO $DB_USER;"

# Create project directories
echo "Creating project directories..."
mkdir $PROJECT_NAME && cd $PROJECT_NAME
mkdir $FRONTEND_DIR $BACKEND_DIR

# Set up Django backend
echo "Setting up Django backend..."
cd $BACKEND_DIR
python3 -m venv venv
source venv/bin/activate
pip install django djangorestframework psycopg2-binary

# Initialize Django project
django-admin startproject $PROJECT_NAME .
python manage.py startapp api

# Update Django settings for PostgreSQL
SETTINGS_FILE="$PROJECT_NAME/settings.py"
sed -i "s/'ENGINE': 'django.db.backends.sqlite3'/'ENGINE': 'django.db.backends.postgresql'/" $SETTINGS_FILE
sed -i "s/'NAME': BASE_DIR / 'db.sqlite3'/'NAME': '$DB_NAME',\n        'USER': '$DB_USER',\n        'PASSWORD': '$DB_PASSWORD',\n        'HOST': 'localhost',\n        'PORT': ''/" $SETTINGS_FILE

# Apply migrations
python manage.py migrate

# Set up React frontend
echo "Setting up React frontend..."
cd ../$FRONTEND_DIR
npx create-react-app .
npm install axios

# Install CORS in Django and configure
echo "Installing and configuring CORS..."
cd ../$BACKEND_DIR
pip install django-cors-headers
echo "INSTALLED_APPS += ['corsheaders']" >> $SETTINGS_FILE
echo "MIDDLEWARE.insert(1, 'corsheaders.middleware.CorsMiddleware')" >> $SETTINGS_FILE
echo "CORS_ALLOW_ALL_ORIGINS = True" >> $SETTINGS_FILE

# Run backend and frontend servers
echo "Starting backend and frontend servers..."
cd ../$BACKEND_DIR
gnome-terminal -- bash -c "source venv/bin/activate && python manage.py runserver; exec bash"
cd ../$FRONTEND_DIR
gnome-terminal -- bash -c "npm start; exec bash"

echo "Setup complete! Django backend, React frontend, and PostgreSQL database are ready."

Usage

1. Save the script as setup_django_fullstack.sh.


2. Make it executable:

chmod +x setup_django_fullstack.sh


3. Run the script:

./setup_django_fullstack.sh



Features

1. Django Backend: Creates a Django project and an api app with PostgreSQL integration.


2. React Frontend: Sets up a React app in a separate directory.


3. PostgreSQL Database: Automatically creates a database and user with necessary privileges.


4. CORS Configuration: Enables communication between the frontend and backend.



You can further customize the script to include additional features as needed.

