# Ice Awareness


## Local Setup
1. `git clone` this repo
2. `cd ice-awareness` 
3. Create `.env.dev` and `.env.dev.db`

This application uses Docker Compose which will build and deploy two containers in dev:
1. The Django application
2. The Postgresql database

You can perform this by running: 

```
docker-compose -f docker-compose.dev.yml up --build
```

To run with clear cache
```
docker-compose -f docker-compose.dev.yml build --no-cache && docker-compose -f docker-compose.dev.yml up -d
```

To view the database
```
psql -U <USERNAME> -W -h localhost -d <DATABASE>
```

The server should be accessible at `http://localhost:8000` and any changes should be reflected.


## Deployment


### How it works
There are three major components:

1. The Nginx proxy: This component receives requests, serves static content, forwards the rest to Django
2. Django App (WSGI Server): This component receives more complicated requests that needs to be processes
3. The Postgres Database (RDS): Stores DB tables

The Nginx proxy and Django app are Dockerized. This application is deployed on AWS Elastic Beanstalk. Here are the steps to deploy this application from scratch:

### Setup
1. Create an AWS account
2. Install the eb cli tool
3. `eb init` will enter interactive mode and save to `.elasticbeanstalk`
    - Region: `1) us-east-1 : US East (N. Virginia)`
    - Platform: Do not select Python, instead select `3) Docker` and `Docker running on 64bit Amazon Linux 2`
    - Do you wish to continue with CodeCommit? (Y/n): NO
    - Do you want to set up SSH for your instances?: YES
4. Create an environment: 
    ```
    eb create <environment_name> --database.engine postgres \
                             --database.username <username> \
                             --database.password <password> \
                             --database.instance <ex db.t3.micro> 
    ```
    - We specify a database to create through the CLI and need to specify the instance type for [compatibility](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html#PostgreSQL.Concepts.General.DBVersions)
    - See the docs for a full list of [options](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-create.html) 
5. Set prod env vars:
    ```
    eb setenv ENV=PROD \
            DEBUG=0 \
            SECRET_KEY=YOUR_SECRET_KEY \
            DJANGO_ALLOWED_HOSTS="localhost 127.0.0.1 [::1] YOUR_DOMAIN_HERE" \
            DOMAIN=YOUR_DOMAIN \
            DB_ENGINE=django.db.backends.postgresql
    ```

After the website is deployed there are a few more steps
1. Create a superuser
    - `eb ssh`
    - Find the web container name: `sudo docker ps`
    - `sudo docker exec -it <web container name> bash` 
    - `python manage.py createsuperuser`
2. Set up Google Allauth ([reference](https://www.section.io/engineering-education/django-google-oauth/))
    - Create OAuth 2.0 Creds [Google Console](https://console.cloud.google.com/apis/credentials)
    - Log in to admin dashboard and add social account with creds

## Useful Commands
```
# Connect to a specific environment (if you are not already)
eb use <environment name>

# View logs
eb logs
docker logs <container name>

# SSH into eb environment
eb ssh

# Deploy changes (picks up local commits)
eb deploy
```

Github Commands

```
# Save working directory
git stash

# Change branches
git checkout <branch_name>

# Checkout new branch 
git checkout -b <branch_name>

# Pop saved stash
git stash pop
```

## Troubleshooting

1. Clip not loading "An Error Occurred": Turn off ghostery or other chrome plugins
