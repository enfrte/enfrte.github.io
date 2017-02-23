# Setting up CodeIgniter to run on Heroku

**Note:** 
 * This does not include a database setup.
 * I've used diylingo as an example project name. 
 * I assume you've installed heroku cli and composer.

Grab a copy of the latest codeigniter and navigate to the root folder from the terminal

    cd diylingo/CodeIgniter-3.1.3/

Login to heroku

    heroku login

Add git 

    git init

Identify heroku as the remote repository for git to use

    heroku create diylingo

In codeigniter, open config.php and add the heroku address, outputted from the ```create``` command, to 

    $config['base_url'] = 'https://diylingo.herokuapp.com/';

Because this will be detected as a php app and there is a composer.lock file which demands you satisfy all dependencies, you'll need to run 

    composer update

Now prepare the files for upload

    git add .

    git commit -m "initial commit"

Push your code to the remote heroku git repository

    git push heroku master

Check the result

    open heroku

You should have the CodeIgniter welcome page opened for you in the browser. 

