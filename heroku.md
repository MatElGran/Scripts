# Heroku

## DB Plan migration

- Setup environment variables

```shell
export PLAN_NAME=standard-0
export APP_NAME=pando2-production
```

- Provision new database

```shell
heroku addons:create heroku-postgresql:"$PLAN_NAME" --app "$APP_NAME"
```

- Wait for new database to be up

```shell
heroku pg:wait --app "$APP_NAME"
```

```shell
export NEW_DATABASE_URL=HEROKU_POSTGRESQL_ORANGE_URL
```

- Enter maintenance mode

```shell
heroku maintenance:on --app "$APP_NAME"
```

- Copy data to target database

```shell
heroku pg:copy DATABASE_URL "$NEW_DATABASE_URL" --app "$APP_NAME"
```

- Promote new DB

```shell
heroku pg:promote "$NEW_DATABASE_URL" --app "$APP_NAME"
```

- Save Old DB new alias, provided by previous command output

```shell
export OLD_DATABASE_URL=HEROKU_POSTGRESQL_OLIVE_URL
```

- Exit maintenance mode

```shell
heroku maintenance:off --app "$APP_NAME"
```

- Destroy old DB

```shell
heroku addons:destroy "$OLD_DATABASE_URL"  --app "$APP_NAME"
```

## Import heroku DB to local server

- Setup environment variables

```shell
export APP_NAME=pando2-production
export LOCAL_DB_HOST=localhost
export LOCAL_DB_USER=postgres
export LOCAL_DB_NAME=pando2_development

```

- Create Heroku DB backup

```shell
heroku pg:backups:capture --app "$APP_NAME"
```

- Download Heroku DB backup

```shell
heroku pg:backups:download --app "$APP_NAME"
```

- Import downloaded dump in local DB

```shell
pg_restore --verbose --clean --no-acl --no-owner -h "$LOCAL_DB_HOST" -U "$LOCAL_DB_USER" -d "$LOCAL_DB_NAME" latest.dump
```


## Import heroku DB to heroku DB

You'll need to push dump file to S3 to be able to restore it to a heroku DB 

- Setup environment variables

```shell
export SRC_APP_NAME=pando2-production
export DST_APP_NAME=pando2-staging
export S3_BUCKET_NAME=pando2-maintenance
export DUMP_FILE_NAME=db.dump
```

- Create Heroku DB backup

```shell
heroku pg:backups:capture --app "$SRC_APP_NAME"
```

- Download Heroku DB backup

```shell
heroku pg:backups:download --app "$DST_APP_NAME" --output "$DUMP_FILE_NAME"
```

- Push dump to S3

```shell
aws s3 cp "$DUMP_FILE_NAME" "s3://$S3_BUCKET_NAME/$DUMP_FILE_NAME"
```

- Presign dump and restore to destination DB

```shell
heroku pg:backups:restore "$(aws s3 presign s3://$S3_BUCKET_NAME/$DUMP_FILE_NAME)" DATABASE_URL --app "$DST_APP_NAME"
```

