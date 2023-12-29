kubeadm join 172.29.28.10:6443 --token 0xvuwq.hc6fvbeubko2god8 \
	--discovery-token-ca-cert-hash sha256:78714e38d79cebc3946bf285591888c6c099ec6862635442707c68454befb389 \
	--control-plane --certificate-key a8b9331404d75ca1e8b94072a1474fcab98e5c039129fcd281a867f51cbd97b2

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.29.28.10:6443 --token 0xvuwq.hc6fvbeubko2god8 \
	--discovery-token-ca-cert-hash sha256:78714e38d79cebc3946bf285591888c6c099ec6862635442707c68454befb389 

