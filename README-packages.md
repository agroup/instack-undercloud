instack-undercloud via packages
=============================

1. The user performing all of the installation steps on the virt host needs to
   have password-less sudo enabled. This step is NOT optional, you must create an
   additional user. Do not run the rest of the steps as root.

        sudo useradd stack
        sudo passwd stack  # specify a password
        echo "stack ALL=(root) NOPASSWD:ALL" | sudo tee -a /etc/sudoers.d/stack
        sudo chmod 0440 /etc/sudoers.d/stack
        sudo su - stack
        echo 'export LIBVIRT_DEFAULT_URI="qemu:///system"' >> ~/.bashrc
        source ~/.bashrc

1. Enable the test TripleO copr repository and install instack-undercloud.

        # fedora
        sudo curl -o /etc/yum.repos.d/slagle-openstack-m.repo https://copr.fedoraproject.org/coprs/slagle/openstack-m/repo/fedora-20/slagle-openstack-m-fedora-20.repo
        # rhel
        sudo curl -o /etc/yum.repos.d/slagle-openstack-m.repo https://copr.fedoraproject.org/coprs/slagle/openstack-m/repo/epel-7/slagle-openstack-m-epel-7.repo
        sudo yum -y install https://repos.fedorapeople.org/repos/openstack/openstack-juno/rdo-release-juno-1.noarch.rpm
        # Enable the stage RDO repo
        sudo sed -i 's#repos.fedorapeople.org/repos#rdo-stage.virt.bos.redhat.com#' /etc/yum.repos.d/rdo-release.repo
        sudo yum -y install instack-undercloud
        
1. Complete the initial setup.

        source /usr/libexec/openstack-tripleo/devtest_variables.sh
        tripleo install-dependencies
        tripleo set-usergroup-membership

1. Verify membership in the libvirtd group

         # verify you are in the libvirtd group
         id | grep libvirtd
         # if not, start a new shell to pick it up
         sudo su - stack

1. Create the virtual environment. When the script has completed successfully,
it will output the instack vm's IP address. Use this IP address in the next
step.

         instack-virt-setup

1. ssh as the stack user (password is stack) to the instack vm

1. Download all the files from http://file.rdu.redhat.com/~jslagle/tripleo-images-juno-source/
   to /home/stack. If you already have them downloaded, use rsync/scp/whatever
   to copy them over to the instack vm. The images will be uploaded to glance
   at the end of the install.

1. Once you are ssh'd into the instack vm as the stack user, setup the copr and
RDO stage repo and install instack-undercloud.

		# Fedora
		sudo curl -o /etc/yum.repos.d/slagle-openstack-m.repo https://copr.fedoraproject.org/coprs/slagle/openstack-m/repo/fedora-20/slagle-openstack-m-fedora-20.repo
		# RHEL
		sudo curl -o /etc/yum.repos.d/slagle-openstack-m.repo https://copr.fedoraproject.org/coprs/slagle/openstack-m/repo/epel-7/slagle-openstack-m-epel-7.repo

        # Fedora and RHEL
        sudo yum -y install https://repos.fedorapeople.org/repos/openstack/openstack-juno/rdo-release-juno-1.noarch.rpm
        # Enable the stage RDO repo
        sudo sed -i 's#repos.fedorapeople.org/repos#rdo-stage.virt.bos.redhat.com#' /etc/yum.repos.d/rdo-release.repo

		sudo yum -y install instack-undercloud

3. Run the installation script.

        instack-install-undercloud

1. Once the install script has run to completion, copy the files
   `/root/stackrc` and `/root/tripleo-undercloud-passwords` into the stack user's home directory.

         sudo cp /root/tripleo-undercloud-passwords .
         sudo cp /root/stackrc .

That completes the Undercloud install. To proceed with deploying and using the
Overcloud see [Overcloud-source](Overcloud-source.md).
