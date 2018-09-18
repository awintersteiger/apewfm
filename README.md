# apeWFM

a simple to use, free workforce management software for teams. Please note, apeWFM has been open sourced based on an
internal project and is currently work in progress. While it works and is being used for years now there are still
some issues as not all settings/data etc. have been made accessible via UI (as they are custom).

apeWFM has been developed by Andreas Wintersteiger in the last years since 2009, most of the time in his spare time.
It has been used at Objectbay, the company founded by the author. Others have contributed to the code as a part of
their employment (see below for details).

## Getting Started

apeWFM is being devloped using Ruby on Rails, MySQL and Docker. You can deploy a local setup based on the source code
or download a docker image (to come).

### Prerequisites

1. Install [Docker](https://docs.docker.com) on your machine 
2. clone the repository

```
git clone 
```

#### Initial Setup

1. Edit the docker-compose files and make neccessary adjustments for your networking infrastructure
1. Edit the docker files (Dokerfile.) and make neccessary adjustments for your networking infrastructure

#### Start the docker containers

Now you just need to start your Docker containers. We use Docker Compose to start all the needed containers. Docker Compose is configured through the `docker-compose.yml` file in your Rails root path. WFM consists of 3 different services (containers):

<br>

- `web`: WFM application itself
- `db`: WFM MySQL database
- `chrome`: Chrome browser for the autmated tests

<br>
First you will need to build the images with

```
docker-compose build
```
<br>
Then create the Docker containers and network with:
```
docker-compose up --no-start
```

Run 
```
docker ps -a
```
to see if all containers were built. You should see something like this in your terminal:


| CONTAINER ID | IMAGE | STATUS  | PORTS  |  NAMES  |
|-----------------|-------------|---------------|---------------|---------------|
| 5efadsfd2 | wfmapp_web  | Exited 5 seconds ago     | 0.0.0.0:3006->3000/tcp    | wfm-web   |
| dfsdfse33 | mysql:5.7  | Exited 5 seconds ago      | 0.0.0.0:3307->3306/tcp    | wfm-db   |
| 324sdfs3e | selenium/standalone-chrome-debug  | Exited 5 seconds ago      | 4444/tcp, 5900/tcp    | wfm-selenium   |

Now run the containers with
```
docker-compose start
```
All three containers must be up and running.

With `docker ps` you can check if the containers are currently running.

You might need to run `docker exec wfm-web rails db:migrate` to run the Rails migrations.

Access the application in your browser with <http://localhost:3006>.
<br><br>

Congratulations. You now have a running apeWFM copy on your local machine.
<br>

#### When to build Image again

Whenever you change the Dockerfile or the Gemfile you need to rebuild the Rails image again.
```
docker-compose build
```

if only the Gems needs to be updated, you can run
```
docker exec -it wfm-web bundle install
```
only also.



## Running the tests

We use Cucumber in combination with Capybara for acceptance tests and Minitest for unit and integration tests.
The Database is populated with the help of Rails fixtures. The test database will be truncated/cleaned in between each test case.

Run acceptance tests:

```
docker exec -it wfm-web cucumber RAILS_ENV=test
```

Run unit and integration tests:

```
docker exec -it wfm-web rails test
``` 

All cucumber tests have to be tagged with a javascript tag (@javascript).
To debug or interact with your system tests, you can use a VNC Client and connect to the selenium container with <vnc://localhost:5900>.

Example:

```ruby
@javascript
Scenario: Log in as an employee with correct credentials
  And I enter correct credentials in the login form
  And I click the login button
  Then I am logged in and I should see the the welcome message on the dashbord page
``` 

## Accessing Development Database

You can access the database with the following settings:

```
Host: 127.0.0.1
Username: wfm_admin
Password: (see the database.yml file for the password)
Database: wfm_development
Port: 3307
```

## Deployment

We use capistrano to deploy and there are three deployment environments as an example, you find them at
```
/config/deploy/production.rb
/config/deploy/test.rb
/config/deploy/uat.rb
```
UAT stands for User Acceptance Testing, the test environment is used to execute automated testing. If you need more environments or different names, use these a template.

1. change the server and user attributes in line 13 of those files.
1. make sure these user names have corresponding ssh keys in the shell available so they can log in to these machines
1. configure rbenv on the target machines
1. set the environment variable WFM_DATABASE_PASSWORD on the target machine (see database.yml for details)

## Installing

1. to set local environment vars for above environments, change the following files:
1.1 /config/local_env_production.yml
1.1 /config/local_env_uat.yml
1. we use Redis for notifications with action cable, the redis servers run in a separate docker container, 
you might need to adjust /config/cable.yml
1. apeWFM uses nginx to serve static files, please adjust
1.1 /conf/nginx_prod.conf
1.1 /conf/nginx_uat.conf
1. Create a new secret base for Rails CSRF tokens and save them into config/secrets.yml

## Initializing with data

### Seeds and Custom Seeds
You might want to change the global settings for the app, there is currently no UI for this. The easiest way is to edit the custom seed file. you can find it in /db/custom_seed/global_settings.rb
Please visit the technical documentation to see how they work.

The same goes for other base data, such as
* function codes
* holidays and religions (we currently provide only data for Austria, insert your own here)
* special leave entitlements 
* travel allowances are only a subset and only for Austria, you should add yours here (Country and refund)
* apeWFM comes with a set of example work time models, you will need to change/add yours here

For all the data and how to add them correctly, please consult my technical documentation.

### Seeds

/db/seeds.rb is run whenever a container is build or you execeute a rails db:seed or similar command.
Upon initial start you might want to change your seed data in line 10 to change your company main address. All other date should not be changed unless you understand the data model and the rake tasks that build the data needed for a successful installation.


### Import a full DB at container build time

You can import an existing database for any deployment, just put the copy into the `/db/imports` folder. The file can have any name, but must be a mysql executable .sql file. The SQL file will be imported into the database everytime you rebuild your MySQL container but not when you restart it. 

### Initial Admin User

Upon creation the setup creates an admin user, that can login with username "admin" and password "admin".
This user is having all roles and you can start adding your employees, ideally you either change this user to your first employee/user having admin rights or finally delete this user again.

### Images / Uploads

Employee avatars are stored in /public/avatars along with the employee id numbers. Never change those directly, users can upload their avatars in their profile setting.

There is also an import and upload folder in /public that is used for different uploads such as doctor certificates etc.

### Company logo for the reports

please manually replace the contents of the file /public/reports_logo.png with your logo.

## Reporting bugs

As mentioned before, apeWFM is currently still in development, there are many bugs and we have our bugtracker at the projects home page. you can report bugs there also.

## Built With

* [Ruby On Rails](https://rubyonrails.org/) - The web framework used
* [MySQL](https://www.mysql.com/) and finally [MariaDB](https://mariadb.org/) when switched to Docker
* [Docker](https://www.docker.com/) - containerization
* other great tools such as nginx, capistrano, capybara and lots of Gems.

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Authors

* **Andreas Wintersteiger** - *Initial and main work* - [awintersteiger](https://github.com/awintersteiger)
* Hadi Samadi - major contributions at Objectbay (2017-2018)
* Christina Grafeneder - contributions at Objectbay (2017-2018)
* Anna Ronacher - contributions at Objectbay (2017-2018)

See also the list of [contributors](https://github.com/awintersteiger/apewfm/graphs/contributors) who participated in this project.

## License

This project is licensed under GNU Affero General Public License (AGPL).

## Acknowledgments

* Thanks go to the employees at Objectbay for testing new releases and finding bugs
* Objectbay has resigned from claiming any rights of the contributions made by their employees
* A few organisations used apeWFM in very early stages and reported bugs - thanks to you guys!
