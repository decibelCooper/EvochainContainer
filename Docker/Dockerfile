# Author: David Blyth
# Description: Docker build merging the FPaDSim chain from Sergei Chekanov
#     with long-term EIC efforts at ANL

FROM dbcooper/arch:2017-05-02

# Set up basic environment
RUN pacman -S --noconfirm \
	sed \
	sudo

RUN useradd -m -G wheel fpadsimuser && \
    sed -i.bak 's/# \(%wheel ALL=(ALL) NOPASSWD: ALL\)/\1/' /etc/sudoers && \
    mkdir /fpadsim && \
    chown fpadsimuser:fpadsimuser /fpadsim

USER fpadsimuser
WORKDIR /fpadsim

CMD /bin/bash -l

# ROOT
RUN sudo pacman -S --noconfirm \
	awk \
	base \
	base-devel \
	binutils \
	cmake \
	fakeroot \
	fftw \
	gcc \
	git \
	glu \
	grep \
	gsl \
	gzip \
	make \
	python2 \
	libx11 \
	libxft \
	libxpm && \
	sudo pacman -R --noconfirm linux

ENV ROOT_VERSION 6-08-06

RUN git clone https://github.com/root-project/root.git && \
	cd root && \
	git checkout tags/v$ROOT_VERSION && \
	cd .. && \
	mkdir build && \
	cd build && \
	cmake ../root \
	    -Dcxx14=ON \
	    -Dgdml=ON \
	    -Dgsl_shared=ON \
	    -Dmathmore=ON \
	    -Dminuit2=ON \
	    -Dopengl=ON && \
	make -j30 && \
	sudo make install && \ 
	cd .. && \
	rm -rf build root

RUN sudo bash -c 'echo ". /usr/local/bin/thisroot.sh" > /etc/profile.d/ROOT.sh' && \
        sudo chmod +x /etc/profile.d/ROOT.sh

# LCIO
ENV	LCIO_VERSION 02-08
ENV	LCIO_DIR /opt/LCIO

RUN git clone https://github.com/iLCSoft/LCIO.git /opt/LCIO && \
        cd $LCIO_DIR && \
	git checkout tags/v$LCIO_VERSION && \
	cd ~/ && \
	mkdir build && \
	cd build && \
	CXXFLAGS=-std=c++14 cmake $LCIO_DIR \
	    -DBUILD_ROOTDICT=ON && \
	make -j30 && \
	make install && \
	cd ../ && \
	rm -rf build

RUN sudo bash -c "echo 'export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:$LCIO_DIR/lib' >> /etc/profile.d/LCIO.sh" && \
	sudo chmod +x /etc/profile.d/LCIO.sh

# CLHEP
RUN sudo pacman -S --noconfirm \
	wget \
        xerces-c

ENV CLHEP_VERSION 2.3.4.4

RUN wget -q http://proj-clhep.web.cern.ch/proj-clhep/DISTRIBUTION/tarFiles/clhep-$CLHEP_VERSION.tgz -O clhep.tgz && \
	tar -xzf clhep.tgz && \
	mv $CLHEP_VERSION/CLHEP ./ && \
	rm -rf $CLHEP_VERSION && \
	mkdir build && \
	cd build && \
	CXXFLAGS=-std=c++14 cmake ../CLHEP && \
	make -j30 && \
	sudo make install && \
	cd .. && \
	rm -rf build CLHEP clhep.tgz

# GEANT4
RUN sudo pacman -S --noconfirm \
        xerces-c

ENV GEANT4_VERSION 10.3.1

RUN git clone https://github.com/Geant4/geant4.git && \
	cd geant4 && \
	git checkout tags/v$GEANT4_VERSION && \
	cd .. && \
        mkdir build && \
        cd build && \
        cmake ../geant4 \
	    -DGEANT4_BUILD_CXXSTD=14 \
	    -DGEANT4_INSTALL_DATA=ON \
	    -DGEANT4_USE_GDML=ON \
	    -DGEANT4_USE_SYSTEM_CLHEP=ON && \
        make -j30 && \
        sudo make install && \
        cd .. && \
        rm -rf build geant4

RUN sudo bash -c 'echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/lib64" >> /etc/profile.d/geant4.sh' && \
        sudo bash -c 'echo ". /usr/local/bin/geant4.sh" > /etc/profile.d/geant4.sh' && \
        sudo chmod +x /etc/profile.d/geant4.sh

# DD4hep
RUN sudo pacman -S --noconfirm \
        boost

ENV DD4HEP_DIR /opt/DD4hep

RUN git clone https://github.com/AIDASoft/DD4hep.git && \
	mkdir build && \
	cd build && \
	CXXFLAGS=-std=c++14 cmake ../DD4hep  \
	    -DBUILD_TESTING=ON \
	    -DCMAKE_INSTALL_PREFIX=$DD4HEP_DIR \
	    -DDD4HEP_USE_CXX14=ON   \
	    -DDD4HEP_USE_GEANT4=ON \
	    -DDD4HEP_USE_GEAR=OFF   \
	    -DDD4HEP_USE_LCIO=ON   \
	    -DDD4HEP_USE_PYROOT=ON \
	    -DDD4HEP_USE_XERCESC=ON && \
	make -j30 && \
	make install && \
	cd .. && \
	rm -rf build DD4hep

