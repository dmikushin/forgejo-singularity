## Preparation

Install singularity-compose via Git:

```
python3 -m venv ./venv
source ./venv/bin/activate.fish
pip install git+https://github.com/singularityhub/singularity-compose.git
```

Create folders that shall be mounted into containers:

```
mkdir forgejo
mkdir mysql
```

## Deployment

```
singularity-compose up -d
```

