# Linux Server Configuration 

## Objective: 
A baseline installation of a Linux server using [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances) and prepare it to host [EDotTech Shop](https://github.com/elhammj/EDotTech) web applications.

## Overview:
This project emphasizes how to access, secure, perform the initial configuration of a bare-bones Linux server and how to install and configure a web and database server and actually host a web application.

## How to start?
1. Sign up [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/)
2. Create Ubuntu Linux server instance by choosing:
	* OS Only
	* Ubuntu 16.04 LTS 
3. Choose your instance plan:
	* The lowest membership is enough for this project
4. Give your instance a hostname
5. Wait for it to start up
6. Once it runs:
	* Copy your public id

## Follow the following steps to configure your instance:
1. Launch Virtual Machine and SSH into the Server
	* In your local machine:
		* Download the private key form Networking part which starts with "LightsailDefaultKey" and with ".pem" extention, rename the file to "Lightsail_Key.rsa " and save it in `~/.ssh/`
		* Change the permission: `chmod 600 ~/.ssh/Lightsail_key.rsa`
		* SHH to your serve rby using Ubuntu User `ssh -i ~/.ssh/Lightsail_Key.rsa -p 2200 ubuntu@54.93.239.136`



### Start the application
1. Access the app through [http://54.93.239.136.xip.io](http://54.93.239.136.xip.io)

## Pages:
1. **Categories**: run in `http://54.93.239.136.xip.io/categories/`
- It shows all the categories and latest 5 items
- Avaliable for both public users and registered users 
2. **Items for a category**: run in `http://54.93.239.136.xip.io/<int:category_id>/items`
- It shows all the items under a selected category
- Avaliable only for registered users
3. **Item's Details**: run in `http://54.93.239.136.xip.io/<int:category_id>/items/<int:item_id>/`
- It shows item's details
- Avaliable only for registered users
4. **Add a new item**: run in `http://54.93.239.136.xip.io/<int:category_id>/items/new/`
- User can add a new item
- Avaliable only for registered users
5. **Edit an item**: run in `http://54.93.239.136.xip.io/<int:category_id>/items/<int:item_id>/edit/`
- User can edit item's info
- Avaliable only for registered users
6. **Delete an item**: run in `http://54.93.239.136.xip.io/<int:category_id>/items/<int:item_id>/delete/`
- User can delete an item
- Avaliable only for registered users
7. **Login Page**: run in `http://54.93.239.136.xip.io/login/`
- User can login by using Google

## JSON Endpoints
### Access JSON pages
`http://54.93.239.136.xip.io/catalog/JSON`
- It has list of categories and its items.

`http://54.93.239.136.xip.io/categories/JSON`
- It has list of all catgeories. 

`http://54.93.239.136.xip.io/categories/<int:category_id>/items/JSON`
- It has list of items under a specific catalog.

`http://54.93.239.136.xip.io/categories/<int:category_id>/items/<int:item_id>/JSON`
- It has item's details.

