# Digital Pianism Test Framework

A simple test framework module that can be used to create unit and integration tests on Magento 1.

## Add required-dev test modules to your project

      "require-dev": {
        "onestic/testframework": "0.1.*",
        "phpunit/phpunit": "^6.0"
      },

## Prepare your module for your tests

 - Create a `Test` folder under your module folder
 
 - Copy _/dev/tests/integration/phpunit.xml.sample_ to _/dev/tests/integration/phpunit.xml_ and update it with your module paths if vendor isn't Onestic.
 
## Controller test sample

Here is a sample of a controller test:

Under your `Test` folder create a `MyTest.php` (your test files must be nammed accordingly to the declaration in your `phpunit.xml.dist` file, in the example above we declared a `Test.php` suffix):

    <?php

    class Vendor_Module_Test_MyTest extends \PHPUnit_Framework_TestCase {

        public function setUp()
        {
            // Stub response to avoid headers already sent problems
            $stubResponse = new \DigitalPianism_TestFramework_Controller_HttpResponse();
            Mage::app()->setResponse($stubResponse);

            // Possible parameter
            // Mage::app()->getRequest()->setParam('myparameter', 'myvalue');

            // Use the controller helper
            $controllerTestHelper = new \DigitalPianism_TestFramework_Helper_ControllerTestHelper($this);

            // Dispatch a GET request
            $controllerTestHelper->dispatchGetRequest('route', 'controller', 'action');
            // Dispatch a POST request
            //$controllerTestHelper->dispatchPostRequest('route', 'controller', 'action');
        }

        public function testSomething()
        {
            // Get the body
            $body = Mage::app()->getResponse()->getBody(true);

            // Get the headers
            $headers = Mage::app()->getResponse()->getHeaders();

            // Get a block
            $block = Mage::app()->getLayout()->getBlock('block_name');

            // Do your tests here
        }
    }

## Using test doubles

As you may have noticed, the `bootstrap.php` injects a custom instance of the config class `DigitalPianism_TestFramework_Model_Config`

This class declares several new methods:

 - `setModelTestDouble` for model test doubles
 - `setResourceModelTestDouble` for resource model test doubles
 - `setHelperTestDouble` for helper test doubles

To use test doubles you can do the following in your `setUp` method.

Let's say you want to check what template ID is assigned to `Mage_Core_Model_Email_Template_Mailer` when the new customer account email is sent:

    $mailer = Mage::getModel('core/email_template_mailer');
    Mage::getConfig()->setModelTestDouble('core/email_template_mailer', $mailer);

    $customer->sendNewAccountEmail();

    $this->assertSame($expectedEmailTemplateId, $mailer->getTemplateId());

## Fixtures

You can create fixtures programmatically in your code.

A good recommendation to avoid having to manually delete your fixtures is to call `Mage::getSingleton('core/resource')->getConnection('core_write')->beginTransaction();` in the `setUp` method and then call `Mage::getSingleton('core/resource')->getConnection('core_write')->rollBack();` in the `tearDown` method

