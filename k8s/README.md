# AddressBook 
![ScreenRecording2024-03-08at1 00 11PM-ezgif com-video-to-gif-converter](https://github.com/kanchan78/AddressBook/assets/65346420/46dda36c-e225-43e7-b8e3-b7cce21cdfc2)



This is a simple CRUD application which helps you to manage your address book.

## Description

AddressBook is a Web Development project for practice purposes. It's a little maven webapp where you can easily create, search, update and delete contacts.

#### Unique features :
* Multiple records can be added through CSV file. [sample upload file](https://docs.google.com/spreadsheets/d/1iFew9bbMiBQdCDlhH689cOFc6vWG5z-IrtaZyyGamWU/edit?usp=sharing)
* Sort the records by any column.
* Search record.

It has been made with:
* Java, using Java Servlets as controllers, JavaBeans as model and JavaServer Pages as view (MVC pattern)
* MySQL
* HTML5 & CSS3
* Bootstrap in order to make it mobile-first and responsive
* JavaScript, jQuery and AJAX

## Getting Started

### Dependencies

* Maven
* JDK 1.8
* Tomcat 8
* Mysql 

### Deployment with Maven

AddressBook can be deployed with Maven or manually with the .war file.

* Clone the repository
* Move in /AddressBook dir and run below command
* Visit localhost and enjoy AddressBook!
  http://localhost:8080/addressBook-2.0/
* Refer to AddressBook.sql for mysql script
```
git clone https://github.com/kanchan78/AddressBook.git
cd AddressBook
mvn clean tomcat7:run-war
```

### Contributing
Contributions are welcome! Fork the repository and create a pull request with your changes.

* Made with ❤ 
* Don't forget to star it 🌟
