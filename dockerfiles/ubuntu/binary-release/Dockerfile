FROM ocaml/opam2:ubuntu-18.04-opam

WORKDIR /home/comby

RUN sudo apt-get update && sudo apt-get install -y \
	libpcre3-dev \
	pkg-config \
	zlib1g-dev \
	m4 \
	libgmp-dev \
    libsqlite3-dev

RUN opam init --no-setup --disable-sandboxing --compiler=4.10.0
RUN eval $(opam env) && opam repository set-url default https://opam.ocaml.org && opam update

COPY Makefile /home/comby/
COPY comby.opam /home/comby/

# make a layer before adding source code but with all deps
RUN eval $(opam env) && opam install . --deps-only -y

COPY dune /home/comby/
COPY dune-project /home/comby/dune-project
COPY docs /home/comby/docs
COPY push-coverage-report.sh /home/comby/
COPY src /home/comby/src
COPY lib /home/comby/lib
COPY test /home/comby/test

RUN sudo chown -R $(whoami) /home/comby
#RUN eval $(opam env) && opam install . --deps-only -y
RUN opam exec -- dune build --profile dev
RUN make comby comby-server benchmark
RUN opam exec -- dune runtest
RUN opam exec -- dune clean
RUN opam exec -- dune build --profile release
RUN echo "\

The binary is now available in <image>:/home/comby/_build/default/src/main.exe.\n\
If your named the image for this build comby-ubuntu-latest:latest then
copy binary to your local directory with:\n\
\n\
docker run -v $PWD:/opt/mount --rm --entrypoint cp comby-ubuntu-latest:latest /home/comby/_build/default/src/main.exe /opt/mount/comby.exe
" 
