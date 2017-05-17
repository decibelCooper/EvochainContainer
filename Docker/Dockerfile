# Author: David Blyth
# Description: Docker build replicating the FPaDSim environment from Sergei Chekanov
#     except for splicing in DD4Hep for simulation.

FROM dbcooper/arch:2017-02-18

# Set up basic environment
RUN pacman -S --noconfirm \
	sed \
	sudo

RUN useradd -m -G wheel fpadsimuser; \
    sed -i.bak 's/# \(%wheel ALL=(ALL) NOPASSWD: ALL\)/\1/' /etc/sudoers;

USER fpadsimuser
WORKDIR /home/fpadsimuser

CMD /bin/bash -l

# ROOT
RUN sudo pacman -S --noconfirm \
	awk \
	base \
	base-devel \
	binutils \
	cmake \
	fakeroot \
	gcc \
	git \
	grep \
	gzip \
	make \
	python \
	wget \
	libx11 \
	libxft \
	libxpm && \
	sudo pacman -R --noconfirm linux

ENV ROOT_VERSION 6.08.06

RUN wget -q https://root.cern.ch/download/root_v$ROOT_VERSION.source.tar.gz && \
	tar -xzf root_v$ROOT_VERSION.source.tar.gz && \
	mkdir build && \
	cd build && \
	cmake ../root-$ROOT_VERSION \
	    -Dcxx14=ON \
	    -Dgdml=ON \
	    -Dgsl_shared=ON \
	    -Dmathmore=ON \
	    -Dminuit2=ON \
	    -Dopengl=ON && \
	make -j4 && \
	sudo make install && \ 
	cd .. && \
	rm -rf root-$ROOT_VERSION build root_v$ROOT_VERSION.source.tar.gz

# DD4hep
ENV DD4HEP_DIR /opt/DD4hep

RUN git clone https://github.com/AIDASoft/DD4hep.git && \
	mkdir build && \
	cd build && \
	cmake ../DD4hep  \
	    -DBUILD_TESTING=ON \
	    -DCMAKE_INSTALL_PREFIX=$DD4HEP_DIR \
	    -DDD4HEP_USE_CXX14=ON   \
	    -DDD4HEP_USE_GEANT4=ON \
	    -DDD4HEP_USE_GEAR=OFF   \
	    -DDD4HEP_USE_LCIO=ON   \
	    -DDD4HEP_USE_PYROOT=ON \
	    -DDD4HEP_USE_XERCESC=ON && \
	make -j4 && \
	make install && \
	cd .. && \
	rm -rf build DD4hep

