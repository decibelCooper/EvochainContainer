# Author: David Blyth
# Description: Docker build intended to replicate the FPaDSim environment
#     created by Sergei Chekanov

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
	python && \
	sudo pacman -R --noconfirm linux

RUN git clone https://aur.archlinux.org/pythia8.git && \
	cd pythia8 && \
	git checkout af8ab26e033e01988346b1e933f033135e1be33b && \
	MAKEFLAGS=-j20 makepkg -si --noconfirm && \
	cd .. && \
	rm -rf pythia8

RUN git clone https://aur.archlinux.org/unuran.git && \
	cd unuran && \
	git checkout f26893309bccabf313895b7aa661b28f2c379950 && \
	MAKEFLAGS=-j20 makepkg -si --noconfirm && \
	cd .. && \
	rm -rf unuran

RUN git clone https://aur.archlinux.org/xrootd-abi0.git && \
	cd xrootd-abi0 && \
	git checkout d75c13211f62fdf6618c7d7399b1ed68482c2e55 && \
	MAKEFLAGS=-j20 makepkg -si --noconfirm && \
	cd .. && \
	rm -rf xrootd-abi0

RUN git clone https://aur.archlinux.org/libafterimage.git && \
	cd libafterimage && \
	git checkout 15cf333b40a64f2b802a7b90ffd55548d78c366c && \
	MAKEFLAGS=-j20 makepkg -si --noconfirm && \
	cd .. && \
	rm -rf libafterimage

RUN git clone https://aur.archlinux.org/root-extra.git && \
	cd root-extra && \
	git checkout 036a35c9076f8b5fd3798c1c7d37ec72a979e249 && \
	echo "set (gdml ON CACHE BOOL \"\" FORCE)" >> settings.cmake && \
	updpkgsums && \
	MAKEFLAGS=-j20 makepkg -si --noconfirm && \
	cd .. && \
	rm -rf root-extra && \
	sudo bash -c 'echo "export ROOTSYS=/" >> /etc/profile.d/root.sh'

# GEANT4
RUN sudo pacman -S --noconfirm \
	xerces-c

RUN git clone https://aur.archlinux.org/qt5-quick1.git && \
	cd qt5-quick1 && \
	git checkout b40a399ab8d7b1a38619e515b240960e0429d722 && \
	sed -i.bak 's/^\( *make\)$/\1 -j30/' PKGBUILD && \
	makepkg -si --noconfirm && \
	cd .. && \
	rm -rf qt5-quick1

RUN git clone https://aur.archlinux.org/soxt.git && \
	cd soxt && \
	git checkout b35aafe46c75da6d20b4dea51ab347daea69fa43 && \
	sed -i.bak 's/^\( *make\)$/\1 -j30/' PKGBUILD && \
	makepkg -si --noconfirm && \
	cd .. && \
	rm -rf soxt

RUN sudo pacman -S --noconfirm \
	wget

ENV G4VERSION 10.03.p01

RUN wget -q http://geant4.cern.ch/support/source/geant4.$G4VERSION.tar.gz && \
	tar -xzf geant4.$G4VERSION.tar.gz && \
	mkdir geant4-build && \
	cd geant4-build && \
	cmake ../geant4.$G4VERSION -DGEANT4_INSTALL_DATA=ON -DGEANT4_USE_GDML=ON && \
	make -j30 && \
	sudo make install && \
	cd .. && \
	rm -rf geant4* && \
	sudo bash -c 'echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/lib64" >> /etc/profile.d/geant4.sh' && \
	sudo bash -c 'echo ". /usr/local/bin/geant4.sh" > /etc/profile.d/geant4.sh' && \
	sudo chmod +x /etc/profile.d/geant4.sh

# Oracle JDK
RUN git clone https://aur.archlinux.org/jdk.git && \
	cd jdk && \
	git checkout eaa4d0bec2f6b573c15c1214c030198215bcb1b2 && \
	makepkg -si --noconfirm && \
	cd .. && \
	rm -rf jdk

# Jas4pp
RUN sudo pacman -S --noconfirm \
	wget

RUN wget -q http://atlaswww.hep.anl.gov/asc/jas4pp/download/current.php -O jas4pp.tgz && \
	tar -xzf jas4pp.tgz -C /opt && \
	rm jas4pp.tgz && \
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/jas4pp.sh" && \
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
	sudo rm -rf ProMC.tgz ProMC && \
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/promc.sh" && \
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
	make -j20 && \
	sudo make install && \
	cd ../ && \
	rm -rf iLCUtil build && \
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/iLCUtil.sh" && \
	sudo bash -c "echo 'export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:$ILCUTIL_DIR/lib' >> /etc/profile.d/iLCUtil.sh" && \
	sudo chmod +x /etc/profile.d/iLCUtil.sh

# LCIO
ENV LCIO_DIR /opt/LCIO

RUN git clone https://github.com/iLCSoft/LCIO.git && \
        cd LCIO && \
	git checkout a0699e2fd9eaf8d9bc03a1ca7d1d0de07c273ff2 && \
	cd ../ && \
	mv LCIO $LCIO_DIR && \
	mkdir build && \
	cd build && \
	CXXFLAGS=-std=c++14 cmake $LCIO_DIR && \
	make -j30 && \
	sudo make install && \
	cd ../ && \
	rm -rf build && \
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/LCIO.sh" && \
	sudo bash -c "echo 'export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:$LCIO_DIR/lib' >> /etc/profile.d/LCIO.sh" && \
	sudo chmod +x /etc/profile.d/LCIO.sh

# SLIC
ENV SLIC_DIR /opt/slic

RUN git clone https://github.com/slaclab/slic.git && \
	cd slic && \
	git checkout 688a8e82551f174aa7bd7b9471bcb264d7164ea0 && \
	echo 'SET( CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -ldl" )' >> CMakeLists.txt && \
	cd ../ && \
	mkdir build && \
	cd build && \
	CXXFLAGS=-std=c++14 cmake ../slic -DINSTALL_DEPENDENCIES=ON -DCMAKE_INSTALL_PREFIX=$SLIC_DIR && \
	make -j30 && \
	CXXFLAGS=-std=c++14 cmake ../slic -DINSTALL_DEPENDENCIES=ON -DCMAKE_INSTALL_PREFIX=$SLIC_DIR && \
	make -j30 VERBOSE=1 && \
	sudo make install && \
	cd ../ && \
	rm -rf slic build && \
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/SLIC.sh" && \
	echo 'export PATH=$PATH:$SLIC_DIR/bin' | sudo bash -c "cat >> /etc/profile.d/SLIC.sh" && \
	echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$SLIC_DIR/lib' | sudo bash -c "cat >> /etc/profile.d/SLIC.sh" && \
	echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$SLIC_DIR/extdeps/gdml/lib' | sudo bash -c "cat >> /etc/profile.d/SLIC.sh" && \
	echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$SLIC_DIR/extdeps/heppdt/lib' | sudo bash -c "cat >> /etc/profile.d/SLIC.sh" && \
	echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$SLIC_DIR/extdeps/lcdd/lib' | sudo bash -c "cat >> /etc/profile.d/SLIC.sh" && \
	sudo chmod +x /etc/profile.d/SLIC.sh

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
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/PandoraPFA.sh" && \
	sudo bash -c "CXXFLAGS=-std=c++14 cmake -DCMAKE_MODULE_PATH=$ROOTSYS/etc/cmake -DPANDORA_MONITORING=ON -DPANDORA_EXAMPLE_CONTENT=OFF \
		-DPANDORA_LAR_CONTENT=OFF -DPANDORA_LC_CONTENT=ON ../" && \
	sudo make -j30 install && \
	echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/PandoraPFA/lib' | sudo bash -c "cat >> /etc/profile.d/PandoraPFA.sh" && \
	sudo chmod +x /etc/profile.d/PandoraPFA.sh

# slicPandora
RUN wget -q http://www.hep.phy.cam.ac.uk/~marshall/slicPandora_Pandora_v02.09.00_patch1.tar.gz -O slicPandora.tar.gz && \
	tar -xzf slicPandora.tar.gz && \
	sudo chown -R root:root slicPandora && \
	sudo mv slicPandora/HEAD /opt/slicPandora && \
	mkdir build && \
	cd build && \
	CXXFLAGS=-std=c++14 cmake /opt/slicPandora -DILCUTIL_DIR=$ILCUTIL_DIR -DLCIO_DIR=$LCIO_DIR -DPandoraPFA_DIR=/opt/PandoraPFA \
		-DPandoraSDK_DIR=/opt/PandoraPFA -DLCContent_DIR=/opt/PandoraPFA -DCMAKE_SKIP_RPATH=1 && \
	make -j30 && \
	sudo make install && \
	cd ../ && \
	sudo rm -rf build slicPandora slicPandora.tar.gz && \
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/slicPandora.sh" && \
	echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/slicPandora/lib' | sudo bash -c "cat >> /etc/profile.d/slicPandora.sh" && \
	echo 'export slicPandora_DIR=/opt/slicPandora' | sudo bash -c "cat >> /etc/profile.d/slicPandora.sh" && \
	sudo chmod +x /etc/profile.d/slicPandora.sh

# promc2lcio
ADD promc2lcio.tar.gz /opt/

RUN sudo chown -R fpadsimuser:fpadsimuser /opt/promc2lcio && \
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/promc.sh" && \
	echo 'export basepath=/opt/promc2lcio/lib; export CLASSPATH=$CLASSPATH:/opt/promc2lcio:${basepath}/freehep-mcfio-2.0.1.jar:${basepath}/lcio-2.4.4-SNAPSHOT-bin.jar:${basepath}/freehep-xdr-2.0.3.jar:/usr/local/promc/java/promc-protobuf.jar:/opt/promc2lcio/lib/browser_promc.jar' | \
		sudo bash -c "cat >> /etc/profile.d/promc.sh" && \
	sudo chmod +x /etc/profile.d/promc.sh

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
	make -j30 && \
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/JNIMathLib.sh" && \
	echo ". /opt/JNIMathLib/bin/JNIMathLib-env.sh" | sudo bash -c "cat >> /etc/profile.d/JNIMathLib.sh" && \
	sudo chmod +x /etc/profile.d/JNIMathLib.sh

# LCSIM
RUN sudo pacman -S --noconfirm \
	maven \
	svn

ENV LCSIM_VERSION 3.4

RUN svn co svn://svn.freehep.org/lcdet/projects/lcsim/tags/lcsim-$LCSIM_VERSION/ /opt/lcsim && \
	cd /opt/lcsim/detector-framework && \
	mvn

RUN cd /opt/lcsim/detector-framework && \
	. /opt/JNIMathLib/bin/JNIMathLib-env.sh && \
	mvn install:install-file -Dfile=$(ls /opt/JNIMathLib/*.jar) -DgroupId=eic -DartifactId=jnimathlib -Dversion=0.1 -Dpackaging=jar && \
	sed -i 's/<\/dependencies>/    <dependency><groupId>eic<\/groupId><artifactId>jnimathlib<\/artifactId><version>0.1<\/version><\/dependency>\n    <\/dependencies>/' pom.xml && \
	sed -i 's/\<Math.acos/eic.jnimathlib.CMath.acos/' ./src/main/java/org/lcsim/detector/solids/GeomOp3D.java && \
	sed -i 's/\(import java\.util\.List;\)/\1\n\nimport eic\.jnimathlib\.CMath;/' ./src/main/java/org/lcsim/detector/solids/GeomOp3D.java

RUN cd /opt/lcsim/detector-framework && \
	. /opt/JNIMathLib/bin/JNIMathLib-env.sh && \
	mvn && \
	cd ../distribution && \
	mvn && \
	echo "#!/bin/bash" | sudo bash -c "cat >> /etc/profile.d/lcsim.sh" && \
	echo "export CLICSOFT=/opt/lcsim; export GEOMCONVERTERDIR=\$CLICSOFT/detector-framework; export GCONVERTER=\$GEOMCONVERTERDIR/target/lcsim-detector-framework-$LCSIM_VERSION-bin.jar" | sudo bash -c "cat >> /etc/profile.d/lcsim.sh" && \
	sudo chmod +x /etc/profile.d/lcsim.sh

