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
	glew \
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

ENV ROOT_VERSION 6-10-00

RUN git clone https://github.com/root-project/root.git && \
	cd root && \
	git checkout tags/v$ROOT_VERSION && \
	cd .. && \
	mkdir build && \
	cd build && \
	cmake ../root \
	    -Dbuiltin_glew=OFF \
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
	sudo bash -c "echo 'export PATH=\$PATH:$LCIO_DIR/bin' >> /etc/profile.d/LCIO.sh" && \
	sudo chmod +x /etc/profile.d/LCIO.sh

# DD4hep
RUN sudo pacman -S --noconfirm \
        boost

ENV DD4HEP_COMMIT c7f31bdc510c1eb8a4041dc4964597b6a495e20d
ENV DD4HEP_DIR /opt/DD4hep

RUN git clone https://github.com/decibelCooper/DD4hep.git && \
	cd DD4hep && \
	git checkout $DD4HEP_COMMIT && \
	cd .. && \
	mkdir build && \
	cd build && \
	. /etc/profile.d/ROOT.sh && \
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

RUN sudo bash -c 'echo ". '$DD4HEP_DIR'/bin/thisdd4hep_only.sh" > /etc/profile.d/DD4hep.sh' && \
        sudo chmod +x /etc/profile.d/DD4hep.sh

# lcgeo
ENV LCGEO_COMMIT 72f221aa21c87c034cc9b96909cfe559915f2715
ENV LCGEO_DIR /opt/lcgeo

RUN git clone https://github.com/iLCSoft/lcgeo.git $LCGEO_DIR && \
	cd $LCGEO_DIR && \
	git checkout $LCGEO_COMMIT && \
	sed -i "s/\<env python\>/env python2/g" $(grep -lr "\<env python\>" .) && \
	sed -i "s/\<bin\/python\>/\/bin\/python2/g" $(grep -lr "\<bin\/python\>" .) && \
	sed -i "9,11d" cmake/thislcgeo.sh.in && \
	cd && \
	mkdir build && \
	cd build && \
	. /etc/profile.d/DD4hep.sh && \
	cmake $LCGEO_DIR \ 
	    -DBUILD_TESTING=OFF && \
	make -j30 install && \
	cd .. && \
	rm -rf build

RUN sudo bash -c 'echo ". '$LCGEO_DIR'/bin/thislcgeo.sh" > /etc/profile.d/lcgeo.sh' && \
        sudo chmod +x /etc/profile.d/lcgeo.sh

# Oracle JDK
RUN git clone https://aur.archlinux.org/jdk.git && \
	cd jdk && \
	git checkout eaa4d0bec2f6b573c15c1214c030198215bcb1b2 && \
	makepkg -si --noconfirm && \
	cd .. && \
	rm -rf jdk

# Jas4pp
RUN wget -q http://atlaswww.hep.anl.gov/asc/jas4pp/download/current.php -O jas4pp.tgz && \
	tar -xzf jas4pp.tgz -C /opt && \
	rm jas4pp.tgz

RUN echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/jas4pp.sh" && \
	sudo bash -c 'echo "source /opt/jas4pp/setup.sh" > /etc/profile.d/jas4pp.sh' && \
	sudo chmod +x /etc/profile.d/jas4pp.sh && \
	source /opt/jas4pp/setup.sh

# ProMC
RUN sudo pacman -S --noconfirm \
	rsync

RUN wget -q http://atlaswww.hep.anl.gov/asc/promc/download/current.php -O ProMC.tgz && \
	tar -xzf ProMC.tgz && \
	cd ProMC && \
	./build.sh && \
	sed -ibak "s/read yn/yn=y/" install.sh && \
	sudo ./install.sh /usr/local && \
	cd ../ && \
	sudo rm -rf ProMC.tgz ProMC

RUN echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/promc.sh" && \
	sudo bash -c 'echo "source /usr/local/promc/setup.sh" > /etc/profile.d/promc.sh' && \
	sudo chmod +x /etc/profile.d/promc.sh

# iLCUtil
ENV ILCUTIL_DIR /opt/iLCUtil

RUN git clone https://github.com/iLCSoft/iLCUtil.git && \
	cd iLCUtil && \
	git checkout 5332ae883348acbf0f32b4439798937fa2428b06 && \
	cd ../ && \
	mkdir build && \
	cd build && \
	cmake ../iLCUtil -DCMAKE_INSTALL_PREFIX=$ILCUTIL_DIR && \
	make -j30 && \
	sudo make install && \
	cd ../ && \
	rm -rf iLCUtil build

RUN echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/iLCUtil.sh" && \
	sudo bash -c "echo 'export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:$ILCUTIL_DIR/lib' >> /etc/profile.d/iLCUtil.sh" && \
	sudo chmod +x /etc/profile.d/iLCUtil.sh

# PandoraPFA
ENV PANDORA_PFA_VERSION v02-09-00

RUN git clone https://github.com/PandoraPFA/PandoraPFA.git && \
	cd PandoraPFA && \
	git checkout $PANDORA_PFA_VERSION && \
	cd ../ && \
	sudo chown -R root:root PandoraPFA && \
	sudo mv PandoraPFA /opt/ && \
	cd /opt/PandoraPFA && \
	sudo mkdir build && \
	cd build && \
	sudo bash -c "CXXFLAGS=-std=c++14 cmake -DCMAKE_MODULE_PATH=$ROOTSYS/etc/cmake -DPANDORA_MONITORING=ON -DPANDORA_EXAMPLE_CONTENT=OFF \
		-DPANDORA_LAR_CONTENT=OFF -DPANDORA_LC_CONTENT=ON ../" && \
	sudo make -j30 install

RUN echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/PandoraPFA.sh" && \
	echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/PandoraPFA/lib' | sudo bash -c "cat >> /etc/profile.d/PandoraPFA.sh" && \
	sudo chmod +x /etc/profile.d/PandoraPFA.sh

# slicPandora
RUN wget -q http://www.hep.phy.cam.ac.uk/~marshall/slicPandora_Pandora_v02.09.00_patch1.tar.gz -O slicPandora.tar.gz && \
	tar -xzf slicPandora.tar.gz && \
	sudo chown -R root:root slicPandora && \
	sudo mv slicPandora/HEAD /opt/slicPandora && \
	mkdir build && \
	cd build && \
	CXXFLAGS=-std=c++14 cmake /opt/slicPandora -DILCUTIL_DIR=$ILCUTIL_DIR -DLCIO_DIR=$LCIO_DIR \
		-DPandoraSDK_DIR=/opt/PandoraPFA -DLCContent_DIR=/opt/PandoraPFA -DCMAKE_SKIP_RPATH=1 && \
	make -j30 && \
	sudo make install && \
	cd ../ && \
	sudo rm -rf build slicPandora slicPandora.tar.gz

RUN echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/slicPandora.sh" && \
	echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/slicPandora/lib' | sudo bash -c "cat >> /etc/profile.d/slicPandora.sh" && \
	echo 'export slicPandora_DIR=/opt/slicPandora' | sudo bash -c "cat >> /etc/profile.d/slicPandora.sh" && \
	sudo chmod +x /etc/profile.d/slicPandora.sh

# promc2lcio
ADD promc2lcio.tar.gz /opt/

RUN sudo chown -R fpadsimuser:fpadsimuser /opt/promc2lcio && \
	cd /opt/promc2lcio && \
	sed -i "441i                        if (status != 1) continue;" promc2lcio.java && \
	sed -i "s/fdata=proev.*/fdata=null\;/" promc2lcio.java && \
	sed -i "s/idata=proev.*/idata=null\;/" promc2lcio.java && \
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/promc.sh" && \
	echo 'export basepath=/opt/promc2lcio/lib; export CLASSPATH=$CLASSPATH:/opt/promc2lcio:${basepath}/freehep-mcfio-2.0.1.jar:${basepath}/lcio-2.4.4-SNAPSHOT-bin.jar:${basepath}/freehep-xdr-2.0.3.jar:/usr/local/promc/java/promc-protobuf.jar:/opt/promc2lcio/lib/browser_promc.jar' | \
		sudo bash -c "cat >> /etc/profile.d/promc.sh" && \
	sudo chmod +x /etc/profile.d/promc.sh && \
	bash -l -c "javac promc2lcio.java"

# lcio2hepsim
ADD lcio2hepsim.tar.gz /opt

RUN sudo chown -R fpadsimuser:fpadsimuser /opt/lcio2hepsim && \
	cd /opt/lcio2hepsim && \
	make clean && \
	make

# fpadsim
RUN echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/fpadsim.sh" && \
	echo 'export FPADSIM=/opt' | sudo bash -c "cat >> /etc/profile.d/fpadsim.sh" && \
	sudo chmod +x /etc/profile.d/fpadsim.sh

# Required tools from Arch repository
RUN sudo pacman -S --noconfirm \
	libxtst \
	zip

# JNIMathLib
RUN git clone https://github.com/decibelCooper/JNIMathLib.git /opt/JNIMathLib &&\
	cd /opt/JNIMathLib && \
	make -j30

RUN echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/JNIMathLib.sh" && \
	echo ". /opt/JNIMathLib/bin/JNIMathLib-env.sh" | sudo bash -c "cat >> /etc/profile.d/JNIMathLib.sh" && \
	sudo chmod +x /etc/profile.d/JNIMathLib.sh

# LCSIM
ENV LCSIM_VERSION 3.9-SNAPSHOT
ENV LCSIM_COMMIT 82bbd9c79fbaddd3e78aafe0a92686e3bea45d21

RUN sudo pacman -S --noconfirm \
        maven

RUN git clone https://github.com/decibelCooper/lcsim.git /opt/lcsim && \
	cd /opt/lcsim && \
	git checkout $LCSIM_COMMIT && \
	cd detector-framework && \
        . /opt/JNIMathLib/bin/JNIMathLib-env.sh && \
        mvn install:install-file -Dfile=$(ls /opt/JNIMathLib/*.jar) -DgroupId=eic -DartifactId=jnimathlib -Dversion=0.1 -Dpackaging=jar && \
        sed -i 's/<\/dependencies>/    <dependency><groupId>eic<\/groupId><artifactId>jnimathlib<\/artifactId><version>0.1<\/version><\/dependency>\n    <\/dependencies>/' pom.xml && \
        sed -i 's/\<Math.acos/eic.jnimathlib.CMath.acos/' ./src/main/java/org/lcsim/detector/solids/GeomOp3D.java && \
        sed -i 's/\(import java\.util\.List;\)/\1\n\nimport eic\.jnimathlib\.CMath;/' ./src/main/java/org/lcsim/detector/solids/GeomOp3D.java && \
        cd .. && \
        mvn -DskipTests && \
	rm -rf ~/.m2

RUN echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/lcsim.sh" && \
        echo "export CLICSOFT=/opt/lcsim; export GEOMCONVERTERDIR=\$CLICSOFT/detector-framework; export GCONVERTER=\$GEOMCONVERTERDIR/target/lcsim-detector-framework-$LCSIM_VERSION-bin.jar" | sudo bash -c "cat >> /etc/profile.d/lcsim.sh" && \
        sudo chmod +x /etc/profile.d/lcsim.sh

# Convenience tools
RUN sudo pacman -S --noconfirm \
        openssh \
	vim

