# EKS + Jenkins (us-east-1 version)

- Region is **us-east-1** (N. Virginia). Your "us-east-1a" is an *Availability Zone* inside this region. Pipelines use **region**, not the AZ.
- If your EKS cluster name is not `vimal-eks`, edit the `aws eks update-kubeconfig` line in the Jenkinsfile.
