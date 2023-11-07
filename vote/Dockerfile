# Utiliser une image de base Python 3.11 sur Alpine pour une taille minimale de l'image.
FROM python:3.11-alpine

WORKDIR /code

COPY requirements.txt requirements.txt

# Installer les dépendances Python définies dans requirements.txt.
# L'option --no-cache empêche l'enregistrement du cache pour réduire la taille de l'image.
RUN pip install --no-cache -r requirements.txt

COPY . .

EXPOSE 8080

CMD ["python", "app.py"]
