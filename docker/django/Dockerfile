# --- BUILD STAGE ---
FROM python:3.9-alpine as builder

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
    
RUN apk add --no-cache \
    build-base \
    python3-dev \
    musl-dev \
    libffi-dev \
    openssl-dev \
    postgresql-dev

WORKDIR /app



COPY ./app/requirements.txt .

RUN pip wheel --wheel-dir=/wheels some-package
RUN pip install --upgrade pip setuptools wheel
RUN pip install --no-cache-dir -r requirements.txt

    # --- FINAL STAGE ---
FROM python:3.11-alpine
    
ENV APP_HOME=/app
WORKDIR $APP_HOME
    
RUN apk add --no-cache libpq
    
COPY --from=builder /wheels /wheels
COPY --from=builder /app/requirements.txt .
RUN pip install --no-cache-dir --no-index --find-links=/wheels -r requirements.txt
    
COPY ./docker/entrypoint.prod.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
    
COPY ./app .
    
RUN python manage.py collectstatic --noinput
    
ENTRYPOINT ["/entrypoint.sh"]
CMD ["gunicorn", "setup.wsgi:application", "--bind", "0.0.0.0:8000"]
    