pipeline {
    agent any
    environment {
        PACKER_ACTION = 'NO'
        TERRAFORM_ACTION = 'DESTROY'
        AMI_ACTION = 'DELETE'
    }

    stages {
        stage('Perform Packer Build') {
            when {
                    expression {
                        env.PACKER_ACTION == 'YES'
                    }
            }
            steps {
                    sh 'pwd'
                    sh 'ls -al'
                    sh 'packer build -var-file packer-vars.json packer.json'
                    sh "tail -2 output.txt | head -2 | awk 'match(\$0, /ami-.*/) { print substr(\$0, RSTART, RLENGTH) }' > ami.txt"
                    sh "echo \$(cat ami.txt) > ami.txt"
                    script {
                        def AMIID = readFile('ami.txt').trim()
                        sh 'echo "" >> variables.tf'
                        sh "echo variable \\\"imagename\\\" { default = \\\"$AMIID\\\" } >> variables.tf"
                    }
            }
        }
        stage('No Packer Build') {
            when {
                    expression {
                        env.PACKER_ACTION != 'YES'
                    }
            }
            steps {
                    sh 'pwd'
                    sh 'ls -al'
                    sh 'echo "" >> variables.tf'
                    sh "echo variable \\\"imagename\\\" { default = \\\"ami-0319a95bbfd9a3a26\\\" } >> variables.tf"
            }
        }
        stage('Terraform Plan') {
            when {
			branch 'master'
            expression {
                env.TERRAFORM_ACTION == 'DEPLOY'
                 }
            }
            steps {
                    sh 'terraform init'
                    sh 'terraform validate'
                    sh 'terraform plan'
            }
        }
        stage('Terraform Apply') {
            when {
			branch 'master'
            expression {
                env.TERRAFORM_ACTION == 'DEPLOY'
                 }
            }
            steps {
                    sh 'terraform init'
                    sh 'terraform apply --auto-approve'
            }
        }
        stage('Terraform State Show') {
            when {
			branch 'master'
            expression {
                env.TERRAFORM_ACTION == 'DEPLOY'
                 }
            }
            steps {
                    sh 'terraform init'
                    sh 'terraform state list'
            }
        }
        stage('Terraform Destroy') {
            when {
			branch 'master'
            expression {
                env.TERRAFORM_ACTION != 'DEPLOY'
                 }
            }
            steps {
                    sh 'terraform init'
                    sh 'terraform destroy --auto-approve'
            }
        }
        stage('Delete AMI') {
            when {
			branch 'master'
            expression {
                env.AMI_ACTION == 'DELETE'
                 }
            }
            steps {
                script {
                        def AMIID = 'ami-0319a95bbfd9a3a26'
                        sh "aws ec2 deregister-image --image-id $AMIID"
                }
            }
        }
    }
}
