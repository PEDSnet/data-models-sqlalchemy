# Data Models SQLAlchemy

[![Circle CI](https://circleci.com/gh/chop-dbhi/data-models-sqlalchemy/tree/master.svg?style=svg)](https://circleci.com/gh/chop-dbhi/data-models-sqlalchemy/tree/master) [![Coverage Status](https://coveralls.io/repos/chop-dbhi/data-models-sqlalchemy/badge.svg?branch=master&service=github)](https://coveralls.io/github/chop-dbhi/data-models-sqlalchemy?branch=master)

SQLAlchemy models and DDL and ERD generation for [chop-dbhi/data-models-service](https://github.com/chop-dbhi/data-models-service) style JSON endpoints.

Web service available at http://dmsa.a0b.io/

## SQLAlchemy Models

In your shell, hopefully within a virtualenv:

```sh
pip install dmsa
```

In python:

```python
import dmsa

# This uses an http request to the data-models-service.
dmsa.add_model_modules()

# Now the data model modules are built and available for import.
from dmsa.omop.v5_0_0.models import Base

for tbl in Base.metadata.sorted_tables:
    print tbl.name
```

Or:

```python
import dmsa
dmsa.add_model_modules()
from dmsa.pedsnet.v2_0_0.models import Person, VisitPayer

print VisitPayer.columns
```

These models are dynamically generated at runtime from JSON endpoints provided by chop-dbhi/data-models-service, which reads data stored in chop-dbhi/data-models. Each data model version available on the service is included in a dynamically generated python module. At the time of writing, the following are available. Any added to the service will use the same naming conventions.

- **OMOP V4** at `omop.v4_0_0.models`
- **OMOP V5** at `omop.v5_0_0.models`
- **PEDSnet V1** at `pedsnet.v1_0_0.models`
- **PEDSnet V2** at `pedsnet.v2_0_0.models`
- **i2b2 V1.7** at `i2b2.v1_7_0.models`
- **i2b2 PEDSnet V2** at `i2b2_pedsnet.v2_0_0.models`
- **PCORnet V1** at `pcornet.v1_0_0.models`
- **PCORnet V2** at `pcornet.v2_0_0.models`
- **PCORnet V3** at `pcornet.v3_0_0.models`

## DDL and ERD Generation

Use of the included Dockerfile is highly recommended to avoid installing DBMS and graphing specific system requirements.

The following DBMS dialects are supported when generating DDL:

- **PostgreSQL** called as `postgresql`
- **MySQL** called as `mysql`
- **MS SQL Server** called as `mssql`
- **Oracle** called as `oracle`

### With Docker:

Retrieve the image:

```sh
docker pull dbhi/data-models-sqlalchemy
```

Usage for DDL generation:

```sh
docker run --rm dbhi/data-models-sqlalchemy dmsa ddl -h
```

Generate OMOP V5 creation DDL for Oracle:

```sh
docker run --rm dbhi/data-models-sqlalchemy dmsa ddl omop 5.0.0 oracle
```

Generate OMOP V5 drop DDL for Oracle:

```sh
docker run --rm dbhi/data-models-sqlalchemy dmsa ddl -d omop 5.0.0 oracle
```

Generate OMOP V5 data deletion DML for Oracle:

```sh
docker run --rm dbhi/data-models-sqlalchemy dmsa ddl -x omop 5.0.0 oracle
```

Usage for ERD generation:

```sh
docker run --rm dbhi/data-models-sqlalchemy dmsa erd -h
```

Generate i2b2 PEDSnet V2 ERD (the image will land at `./erd/i2b2_pedsnet_2.0.0_erd.png`):

```sh
docker run --rm -v $(pwd)/erd:/erd dbhi/data-models-sqlalchemy dmsa erd i2b2_pedsnet 2.0.0 /erd/i2b2_pedsnet_2.0.0_erd.png
```

The `graphviz` graphing package supports a number of other output formats, listed here (link pending), which are interpreted from the passed extension.

### Without Docker:

Install the system requirements (see Dockerfile for details):

- **Python 2.7**
- `graphviz` for ERD generation
- Oracle `instantclient-basic` and `-sdk` and `libaio1` for Oracle DDL generation
- `libpq-dev` for PostgreSQL DDL generation
- `unixodbc-dev` for MS SQL Server DDL generation

Install the python requirements, hopefully within a virtualenv (see Dockerfile for details):

```sh
pip install cx-Oracle            # for Oracle DDL generation
pip install psycopg2             # for PostgreSQL DDL generation
pip install PyMySQL              # for MySQL DDL generation
pip install pyodbc               # for MS SQL Server DDL generation
```

Install the data-models-sqlalchemy python package:

```sh
pip install dmsa
```

Usage for DDL generation:

```sh
dmsa ddl -h
```

Generate OMOP V5 creation DDL for Oracle:

```sh
dmsa ddl omop 5.0.0 oracle
```

Generate OMOP V5 drop DDL for Oracle:

```sh
dmsa ddl -d omop 5.0.0 oracle
```

Generate OMOP V5 data deletion DML for Oracle:

```sh
dmsa ddl -x omop 5.0.0 oracle
```

Usage for ERD generation:

```sh
dmsa erd -h
```

Generate i2b2 PEDSnet V2 ERD (the image will land at `./erd/i2b2_pedsnet_2.0.0_erd.png`):

```sh
mkdir erd
dmsa erd i2b2_pedsnet 2.0.0 ./erd/i2b2_pedsnet_2.0.0_erd.png
```

## Web Service

The web service uses a [Gunicorn](http://gunicorn.org/) server in the Docker container and the Flask debug server locally. It exposes the following endpoints:

- Creation DDL at `/<model>/<version>/ddl/<dialect>/`
- Creation DDL for only `table`, `constraint`, or `index` elements at `/<model>/<version>/ddl/<dialect>/<elements>`
- Drop DDL at `/<model>/<version>/drop/<dialect>/`
- Drop DDL for only `table`, `constraint`, or `index` elements at `/<model>/<version>/drop/<dialect>/<elements>`
- Data deletion DML at `/<model>/<version>/delete/<dialect>/`
- ERDs at `/<model>/<version>/erd/`

### With Docker:

Usage:

```sh
docker run dbhi/data-models-sqlalchemy gunicorn -h
```

Run:

```sh
docker run dbhi/data-models-sqlalchemy  # Uses Dockerfile defaults of 0.0.0.0:80
```

### Without Docker:

Install Flask:

```sh
pip install Flask
```

Usage:

```sh
dmsa start -h
```

Run:

```sh
dmsa start                              # Uses Flask defaults of 127.0.0.1:5000
```
