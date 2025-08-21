# 🚀 KodeKloud Lab -- Terraform Challenge 2

This repository contains the solution for **KodeKloud Terraform Challenge 2**.
In this challenge, we deploy a **LAMP stack** (Linux, Apache, MySQL/MariaDB, PHP) using Terraform with the **Docker provider**.

------------------------------------------------------------------------

## 🗺️ Architecture Diagram

<img width="1036" height="663" alt="image" src="https://github.com/user-attachments/assets/531a03a7-ba98-4716-9f62-e465176e911f" />

------------------------------------------------------------------------

## 🛠️ Deployment Steps

### ✅ Step 1 -- Install Terraform (version 1.1.5) on the `iac-server`

1.  Check if Terraform is installed:

``` bash
terraform --version
```

If no output is returned, Terraform is not installed.

2.  Update the OS, ensure `unzip` is available, then download and
    install Terraform v1.1.5:

``` bash
apt update
apt install unzip -y
curl -L -o /tmp/terraform_1.1.5_linux_amd64.zip https://releases.hashicorp.com/terraform/1.1.5/terraform_1.1.5_linux_amd64.zip
unzip -d /usr/local/bin /tmp/terraform_1.1.5_linux_amd64.zip
```

------------------------------------------------------------------------

### ✅ Step 2 -- Configure Docker Provider

The Docker provider is already configured using **kreuzwerker/docker**.\
Documentation: [Terraform Docker Provider](https://registry.terraform.io/providers/kreuzwerker/docker/latest).

Check the existing provider configuration:

``` bash
cd /root/code/terraform-challenges/challenge2
cat provider.tf
```

Initialize the provider:

``` bash
terraform init
```
------------------------------------------------------------------------
### ✅ Step 3 -- Create Terraform Resources

Use `vi` or any text editor to create the following files under: `/root/code/terraform-challenges/challenge2`

#### 1️⃣ php-httpd-image.tf

``` hcl
resource "docker_image" "php-httpd-image" {
  name = "php-httpd:challenge"
  build {
    path = "lamp_stack/php_httpd"
    tag  = ["php-httpd:challenge"]
    label = {
      challenge = "second"
    }
  }
}
```

#### 2️⃣ mariadb-custom-image.tf

``` hcl
resource "docker_image" "mariadb-image" {
  name = "mariadb:challenge"
  build {
    path = "lamp_stack/custom_db"
    tag  = ["mariadb:challenge"]
    label = {
      challenge = "second"
    }
  }
}
```

#### 3️⃣ mariadb_volume.tf

``` hcl
resource "docker_volume" "mariadb_volume" {
  name = "mariadb-volume"
}
```

#### 4️⃣ private_network.tf

``` hcl
resource "docker_network" "private_network" {
  name       = "my_network"
  attachable = true
  labels {
    label = "challenge"
    value = "second"
  }
}
```

#### 5️⃣ db.tf

``` hcl
resource "docker_container" "mariadb" {
  name     = "db"
  image    = docker_image.mariadb-image.name
  hostname = "db"

  networks_advanced {
    name = docker_network.private_network.id
  }

  ports {
    internal = 3306
    external = 3306
    ip       = "0.0.0.0"
  }

  labels {
    label = "challenge"
    value = "second"
  }

  env = [
    "MYSQL_ROOT_PASSWORD=1234",
    "MYSQL_DATABASE=simple-website"
  ]

  volumes {
    container_path = "/var/lib/mysql"
    volume_name    = docker_volume.mariadb_volume.name
  }
}
```

#### 6️⃣ php-httpd.tf

``` hcl
resource "docker_container" "php-httpd" {
  name     = "webserver"
  image    = docker_image.php-httpd-image.name
  hostname = "php-httpd"

  networks_advanced {
    name = docker_network.private_network.id
  }

  ports {
    internal = 80
    external = 80
    ip       = "0.0.0.0"
  }

  labels {
    label = "challenge"
    value = "second"
  }

  volumes {
    container_path = "/var/www/html"
    host_path      = "/root/code/terraform-challenges/challenge2/lamp_stack/website_content/"
  }
}
```

#### 7️⃣ db_dashboard.tf

``` hcl
resource "docker_container" "phpmyadmin" {
  name     = "db_dashboard"
  image    = "phpmyadmin/phpmyadmin"
  hostname = "phpmyadmin"

  networks_advanced {
    name = docker_network.private_network.id
  }

  depends_on = [
    docker_container.mariadb
  ]

  links = [
    docker_container.mariadb.name
  ]

  labels {
    label = "challenge"
    value = "second"
  }

  ports {
    internal = 80
    external = 8081
    ip       = "0.0.0.0"
  }
}
```

------------------------------------------------------------------------

### ✅ Step 4 -- Deploy and Verify

Deploy with:

``` bash
terraform plan
terraform apply
```

Verify deployment:

-   Access `php-httpd` on port `80`.\ and `phpMyAdmin` on port `8081`.\
-   Or simply use the lab's **check button**.


<img width="1919" height="818" alt="image" src="https://github.com/user-attachments/assets/23b81632-0707-4d58-8125-d6d17bd01e78" />

**🎉 Congratulations -- We have successfully completed Terraform Challenge 2 !**

------------------------------------------------------------------------

## 🧠 Contributions

Contributions, corrections, and improvements are welcome.\
Feel free to open a **pull request** or create an **issue**.

------------------------------------------------------------------------

## ✍️ Author

Made with 💻 by **Nidhal Labri**\
🔗 [LinkedIn](https://www.linkedin.com/in/nidhal-labri/)
