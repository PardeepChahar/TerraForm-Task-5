# TerraForm-Task-5
Write terraform code to configure the following things on AWS - 

📍Launch One VPC in north verginia having name as lwterra

📍 Create EC2 instance in this VPC

📍Launch Webserver using apache 

📍Create a index.html file having content about your skills. 

📍Start the webserver on port 80

📍 Create the snapshot of volume attached with the EC2 instance. 

📍Destroy all the resources created in this task. 

#### Code

# create VPC
resource "aws_vpc" "main_vpc" {
  cidr_block       = "10.0.0.0/16"

  tags = {
    Name = "main-vpc"
  }
}

# create subnet in main_vpc
resource "aws_subnet" "main_subnet" {
  vpc_id     = aws_vpc.main_vpc.id
  cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = true

  tags = {
    Name = "main-subnet"
  }
}

# create internet gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main_vpc.id
  tags = {
    Name = "main_igw"
  }
}

# craete route table
resource "aws_route_table" "main_rt" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "main-rt"
  }
}

# associate route table
resource "aws_route_table_association" "a" {
  subnet_id      = aws_subnet.main_subnet.id
  route_table_id = aws_route_table.main_rt.id
}

#Create secrity group(ssh and http rule) in default vpc
resource "aws_security_group" "allow_ssh" {
  name        = "allow_ssh"
  description = "Allow ssh inbound traffic"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    description      = "ssh from VPC"
    from_port        = 22
    to_port          = 22
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
ingress {
    description      = "Http"
    from_port        = 80
    to_port          = 80
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "allow_ssh"
  }
}

# create ec2 instance
resource "aws_instance" "new-ec2" {
  ami           = "ami-0dc2d3e4c0f9ebd18"
  instance_type = "t2.micro"
  key_name = "pk_aws_key"
  subnet_id      = aws_subnet.main_subnet.id
  security_groups  = [aws_security_group.allow_ssh.id]

}

# add null resource to connect and install & start server
resource "null_resource" "nr1"{
 connection {
    type     = "ssh"
    user     = "ec2-user"
    private_key = file("F:/RHEL/Terraform/pk_aws_key.pem")
    host     = aws_instance.new-ec2.public_ip
  }
  provisioner "remote-exec" {
    inline = [
      "sudo yum  install httpd  -y",
		"sudo  yum  install php  -y",
		"sudo systemctl start httpd",
		"sudo systemctl enable httpd",
		"sudo yum install git -y",
		"sudo git clone https://github.com/vimallinuxworld13/gitphptest.git   /var/www/html"
    ]
  }
}
