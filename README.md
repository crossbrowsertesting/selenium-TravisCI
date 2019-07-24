<h2><strong>Getting Started with Travis CI and CrossBrowserTesting</strong></h2>
<p><em>For this document, we provide example tests located in our <a href="https://github.com/crossbrowsertesting/selenium-TravisCI/tree/master/tests">Travis CI&nbsp;Github Repository</a>.</em></p>
<p><a href="https://travis-ci.org/">Travis CI</a> is a continuous integration tool that lets you automate your development process quickly, safely, and at scale. Through Travis CI's integration with GitHub, every time you commit code, a build is created and automatically run, allowing you to test every commit.</p>
<p>In this guide we will use Travis CI with Github for testing using the <a href="https://www.seleniumhq.org/">Selenium Webdriver</a> and <a href="https://www.python.org/">Python</a> programming language.</p>
<h3>Setting up Travis CI</h3>
<p>1. Navigate to your GitHub account and <a href="https://github.com/new">create a new repository</a>.<img src="http://help.crossbrowsertesting.com/wp-content/uploads/2019/07/create_repository_1.png"></p>
<p>2. Add file tests/test_selenium.py
</p><pre><code>from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.by import By
from selenium import webdriver
import unittest
import requests
import os

class LoginForm(unittest.TestCase):
    def setUp(self):

        # Put your username and authkey below
        # You can find your authkey at crossbrowsertesting.com/account
        self.username = os.environ.get(<span class="pl-s"><span class="pl-pds">'</span>CBT_USERNAME<span class="pl-pds">'</span></span>)
        self.authkey  = os.environ.get(<span class="pl-s"><span class="pl-pds">'</span>CBT_AUTHKEY<span class="pl-pds">'</span></span>)

        self.api_session = requests.Session()
        self.api_session.auth = (self.username,self.authkey)

        self.test_result = None

        caps = {}

        caps['name'] = 'Travis CI Example'
        caps['browserName'] = 'Chrome'
        caps['version'] = '75'
        caps['platform'] = 'Windows 10'
        caps['screenResolution'] = '1366x768'
        caps['username'] = self.username
        caps['password'] = self.authkey

        self.driver = webdriver.Remote(
            desired_capabilities=caps,
            command_executor="http://%s:%s@hub.crossbrowsertesting.com:80/wd/hub"
        )

        self.driver.implicitly_wait(20)

    def test_CBT(self):

        try:
            self.driver.get('http://crossbrowsertesting.github.io/login-form.html')
            self.driver.maximize_window()
            self.driver.find_element_by_name('username').send_keys('tester@crossbrowsertesting.com')
            self.driver.find_element_by_name('password').send_keys('test123')
            self.driver.find_element_by_css_selector('body &gt; div &gt; div &gt; div &gt; div &gt; form &gt; div.form-actions &gt; button').click()

            elem = WebDriverWait(self.driver, 10).until(
                EC.presence_of_element_located((By.XPATH, '//*[@id=\"logged-in-message\"]/h2'))
            )

            welcomeText = elem.text
            self.assertEqual("Welcome tester@crossbrowsertesting.com", welcomeText)

            print("Taking snapshot")
            snapshot_hash = self.api_session.post('https://crossbrowsertesting.com/api/v3/selenium/' + self.driver.session_id + '/snapshots').json()['hash']

            self.test_result = 'pass'

        except AssertionError as e:
            # log the error message, and set the score to "during tearDown()".
            self.api_session.put('https://crossbrowsertesting.com/api/v3/selenium/' + self.driver.session_id + '/snapshots/' + snapshot_hash,
                data={'description':"AssertionError: " + str(e)})
            self.test_result = 'fail'
            raise

    def tearDown(self):
        print("Done with session %s" % self.driver.session_id)
        self.driver.quit()
        # Here we make the api call to set the test's score.
        # Pass it it passes, fail if an assertion fails, unset if the test didn't finish
        if self.test_result is not None:
            self.api_session.put('https://crossbrowsertesting.com/api/v3/selenium/' + self.driver.session_id,
                data={'action':'set_score', 'score':self.test_result})

if __name__ == '__main__':
    unittest.main()
</code></pre>
<p>3. Add file .travis.yml to the new repository
</p><pre><code>sudo: false
language: python
# command to run tests
script: pytest
install:
      - pip install selenium
      - pip install requests
</code></pre>
<h3><strong>Running a test</strong></h3>
<div class="blue-alert">You’ll need to use your Username and Authkey to run your tests on CrossBrowserTesting. To get yours, sign up for a&nbsp;<a href="https://crossbrowsertesting.com/freetrial"><b>free trial</b></a>&nbsp;or purchase a&nbsp;<a href="https://crossbrowsertesting.com/pricing"><b>plan</b></a>.</div>
<p>1. From the <a href="https://travis-ci.org/">Travis CI homepage</a>, <strong>Sign in with Github</strong>&nbsp; <img src="http://help.crossbrowsertesting.com/wp-content/uploads/2019/07/sign_in_with_github_3.png"></p>
<p>2. <a href="https://travis-ci.org/account/repositories">Add your repository</a></p>
<p><img src="http://help.crossbrowsertesting.com/wp-content/uploads/2019/07/add_repository.png"></p>
<p>3. Navigate to repository settings.</p>
<p><img src="http://help.crossbrowsertesting.com/wp-content/uploads/2019/07/repository_settings.png"></p>
<p>4. Save&nbsp;your CBT username and authkey as private environment variables</p>
<p><img src="http://help.crossbrowsertesting.com/wp-content/uploads/2019/07/environment_variables.png"></p>
<p>Congratulations! You have successfully integrated CBT and Travis CI. Now you are ready to see your build start to run from the <a href="https://travis-ci.org/dashboard">Travis CI dashboard</a> and in the <a href="https://app.crossbrowsertesting.com/selenium/results">Crossbrowsertesting app</a>.</p>
<h3>Conclusions</h3>
<p>By following the steps outlined in this guide, you are now able to seamlessly integrate Travis CI and CrossBrowserTesting. If you have any questions or concerns, please feel free to reach out to our <a href="mailto:support@crossbrowsertesting.com">support team</a>.</p>
