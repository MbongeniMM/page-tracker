# Pull a slim python image
FROM python:3.11.2-slim-bullseye AS builder

# patch it with the most recent security updates and bug fixes
RUN apt-get update && \
    apt-get upgrade --yes

# switch to a regular user without administrative privileges
RUN useradd --create-home mbongeni
USER mbongeni
WORKDIR /home/mbongeni

# create a virtual environment and modify its PATH env variable
ENV VIRTUALENV=/home/mbongeni/venv
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

COPY --chown=mbongeni pyproject.toml constraints.txt ./

# install your dependencies
RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir -c constraints.txt ".[dev]"

# copy source code to the image
COPY --chown=mbongeni src/ src/
COPY --chown=mbongeni test/ test/

# run your tests along with linters and other static analysis tools
RUN python -m pip install . -c constraints.txt && \
    python -m pytest test/unit/ && \
    python -m flake8 src/ && \
    python -m isort src/ --check && \
    python -m black src/ --check --quiet && \
    python -m pylint src/ --disable=C0114,C0116,R1705 && \
    python -m bandit -r src/ --quiet && \
    python -m pip wheel --wheel-dir dist/ . -c constraints.txt

FROM python:3.11.2-slim-bullseye

RUN apt-get update && \
    apt-get upgrade --yes

RUN useradd --create-home mbongeni
USER mbongeni
WORKDIR /home/mbongeni

ENV VIRTUALENV=/home/mbongeni/venv
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

COPY --from=builder /home/mbongeni/dist/page_tracker*.whl /home/mbongeni

RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir page_tracker*.whl

# specifying the command to execute inside each new Docker container
CMD ["flask", "--app", "page_tracker.app", "run", \
     "--host", "0.0.0.0", "--port", "5000"]

