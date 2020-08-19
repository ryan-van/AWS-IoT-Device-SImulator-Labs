# IoT SiteWise Lab

In this lab you will work with IoT SiteWise; a managed service that makes it easy to collect, store, organize and monitor data from industrial equipment at scale.

We will be modeling a solar farm. Inside our solar farm, we will include two device types, a solar weather tracker and a solar panel tracker. These two devices will ingest data into IoT SiteWise and we will be building meaningful data graphs out of them.

## Architecture
![Architecture](/images-sitewise/architecture.jpg)

## Prerequisites
* AWS Account
* Device Simulator Setup from Alert Lab

## Step 1 - Create a New Device Type on Device Simulator
We’re going to setup two devices here, solar weather and solar panel.
* Login to our device simulator console from before
* Click on Create a Device Type
    * Name the device: SolarWeather
    * Data Topic: /solar/farm/weather
    * Data Transmission Duration: 3000000
    * Data Transmission Interval: 20000
    * Message Payload:
        * Device ID:
            * Attribute Name: Name
            * Type: Device ID
        * Temperature:
            * Attribute Name: temperature
            * Type: Float
            * Min: 30
            * Max: 110
        * WindSpeed:
            * Attribute Name: windspeed
            * Type: Int
            * Min: 1
            * Max: 60
        * Humidity:
            * Attribute Name: humidity
            * Type: Int
            * Min: 1
            * Max: 300
    * ![Device Sim Weather](/images-sitewise/devicesim-weather.png)
* Create a second Device:
    * Name the device: SolarPanel
    * Data Topic: /solar/farm/panel
    * Data Transmission Duration: 3000000
    * Data Transmission Interval: 20000
    * Message Payload:
        * Device ID:
            * Attribute Name: Name
            * Type: Device ID
        * Energy Output:
            * Attribute Name: energyoutput
            * Type: Float
            * Min: 6
            * Max: 10
        * Cell Temperature:
            * Attribute Name: celltemperature
            * Type: Float
            * Min: 50
            * Max: 129
        * Efficiency:
            * Attribute Name: efficiency
            * Type: Float
            * Min: 15
            * Max: 42
    * ![Device Sim Panel](/images-sitewise/devicesim-panel.png)
* Create Widgets for each device

## Step 2 - Create SiteWise Models
Let’s move onto SiteWise
* Go to the [SiteWise website](https://console.aws.amazon.com/iotsitewise/home) (make sure you’re in the correct region)
* On the left panel, select Models under the Build category
Our architecture is going to follow a Solar Farm overarching two devices in it, a SolarPanel and SolarWeather device. First, let’s create our Solar Panel Model:
* Create Solar Panel Model:
    * Name: Solar Panel
    * Measurement Definitions:
    ![Model Panel Measurement](/images-sitewise/sitewise-model-panel.png)
    * Metric Definitions:
    ![Model Panel Metric](/images-sitewise/sitewise-model-panel-2.png)
* Create Solar Weather Model:
    * Name: Solar Weather
    * Measurement Definitions:
    ![Model Weather Measurement](/images-sitewise/sitewise-model-weather.png)
    * Metric Definitions:
    ![Model Weather Metric](/images-sitewise/sitewise-model-weather-2.png)
* Create Solar Farm Model:
    * Name: Solar Farm
    * Hierarchy Definitions:
    * Metric Definitions:
    ![Model Farm](/images-sitewise/sitewise-model-farm.png)

## Step 3 - Create SiteWise Assets
Now that we have our Models, we can create Assets which are the actual devices based on Models that we previously defined. Access assets by clicking on the left panel and go to Build → Assets.
* Create an asset:
    * Model and Name: Solar Farm
* Create an asset:
    * Model and Name: Solar Weather
* Create an asset:
    * Model and Name: Solar Panel
Let’s tag associate our solar panel and weather assets with our farm:
* Click on the Solar Farm asset and Edit
    * Under “Assets associated to this asset”
    * Hierarchy: Panel; Asset: Solar Panel
    * Hierarchy: Weather; Asset: Solar Weather
We now have to edit each asset to associate a property alias with each.
* Edit the Solar Panel Asset:
    * Under Measurement, add the property aliases:
        * Energy Output: /solar/farm/panel/energy
        * Cell Temperature: /solar/farm/panel/temperature
        * Solar Efficiency: /solar/farm/panel/efficiency
    * ![Asset Panel](/images-sitewise/sitewise-asset-panel.png)
* Edit the Solar Weather Asset:
    * Under Measurement, add the property aliases:
        * Humidity: /solar/farm/weather/humidity
        * Temperature: /solar/farm/weather/temperature
        * WindSpeed: /solar/farm/weather/windspeed
    * ![Asset Weather](/images-sitewise/sitewise-asset-weather.png)

By associating an alias with each one, we can later reference them when setting up our IoT Rules.

## Step 4 - Setup SiteWise Portals and SSO
Now we’re going to create our data portal
* Go to Portals and click on Create Portal
* We’re going to have to set up Single Sign On (SSO) first; if you already have this complete, skip this step
    * Enter the Email address, First name, and Last name for the user that you want as your portal administrator.tv The given email address will receive an email to set a password for the new AWS SSO user. If you want to be the portal administrator, enter your email and name to create an AWS SSO identity to use with your portal. You can create more users later.
    * Choose Create user to create the portal administrator user and enable AWS SSO
    * ![SSO](/images-sitewise/sso.png)
    * All users accessing portals will sign in with SSO
    * \*Note, this can be in any region and can only be done in one region
* Portal Configuration:
    * Portal name: solar-farm
    * Support Contact: you@email.com
    * Permissions: Create and use a new service role
    * Create Portal
* Invite Admins, this is where you invite an email to access the portal
    * Create User:
    * ![Sitewise portal](/images-sitewise/sitewise-portal-create.png)
    * Send invite to selected users (Optional if you want to send your own email)
* Assign users to access portal
    * Can either create a new user here or send invite to previous user/admin
* Your first time logging into the portal will require setting your password.

## Step 5 - Setup SiteWise Graphs
After the portal is created, we can now click on the link given to us to open the portal
![Sitewise login](/images-sitewise/sitewise-portal-login.png)
* Login to the portal
We will begin by creating a project which defines which set of assets we will be using. After that, we can establish our Dashboards and choose what we want to graph
* Create a Project
    * Name: solar-farm
    * Description: Solar Farm Dashboards
* Attach project assets:
    * Under Project Assets, “Go to asset library”
    * Select Solar Farm
    * Click on “Add asset to property”
    * Select existing project - “solar-farm”
* Create a Dashboard for Solar Panel Data
    * Click on Create a Dashboard
    * Select Solar Panel on the right pane
    * Drag and Drop the Properties
    ![Sitewise Dashboard](/images-sitewise/sitewise-portal-dashboard.png)
    * Here, you can change the graphs to add thresholds to easier look at your data
    ![Sitewise Thresholds](/images-sitewise/sitewise-thresholds.png)
    * You can also change the Visualization type and add multiple properties to the same graph
    ![Sitewise Visualizations](/images-sitewise/sitewise-visuialization.png)
    * Save the Dashboard
* Create a Dashboard for Solar Weather
    * Create this however you want, make sure to drag your properties onto the board
    * The size of the graphs can also be changed
    ![Sitewise Dashboard 2](/images-sitewise/sitewise-portal-dashboard-2.png)

## Step 6 - Connect IoT Core with SiteWise
Let’s go back to IoT Core to create a Rule to send data to SiteWise from our Device Simulator
* Create a new Rule (Under Act → Rules)
    * Name: SolarFarm
    * SQL Statement: SELECT * FROM '/solar/farm/+'
        * Here, we use the wildcard ‘+’ for sql rules and allows us to grab from multiple data topics
    * Add a new Action:
        * Select “Send message data to asset properties in AWS IoT SiteWise”:
        * Select By Property Alias:
            * Property Alias: /solar/farm/panel/energy
                * Here, we’re going to enter the alias we made before
            * Time in seconds: ${floor(timestamp() / 1E3)}
                * There are built in functions for getting the current time from IoT Rules, here we call it and transform it to make it work
            * Value: ${energyoutput}
                * The dollar sign bracket notation is used to signal that we are pulling from our IoT Topic, ‘/solar/farm/+’, by specifying the name that we used from Device Simulator (JSON)
            * Data Type: Double
    * ![IoT Core](/images-sitewise/iot-core.png)
    * Select Add Entry:
        * Property Alias: /solar/farm/panel/temperature
        * Time in Seconds: ${floor(timestamp() / 1E3)}
        * Value: ${celltemperature}
        * Data Type: Double
    * Add another entry:
        * Property Alias: /solar/farm/panel/efficiency
        * Time in Seconds: ${floor(timestamp() / 1E3)}
        * Value: ${efficiency}
        * Data Type: Double
    * Root Asset Name:
        * Select the Solar Farm we created
    * Role:
        * Create New Role
        * Name the role: IoTSolarFarmRole
    * We’re going to repeat the same thing for Solar Weather now:
    * Add a new Action:
        * Select “Send message data to asset properties in AWS IoT SiteWise”:
            * First Entry:
                * Property Alias: /solar/farm/weather/humidity
                * Time in Seconds: ${floor(timestamp() / 1E3)}
                * Value: ${humidity}
                * Data Type: Integer
            * Second Entry:
                * Property Alias: /solar/farm/weather/temperature
                * Time in Seconds: ${floor(timestamp() / 1E3)}
                * Value: ${temperature}
                * Data Type: Double
            * Last Entry:
                * Property Alias: /solar/farm/weather/windspeed
                * Time in Seconds: ${floor(timestamp() / 1E3)}
                * Value: ${windspeed}
                * Data Type: Integer
    * Attach the role and update it
* Should end up looking like this:
![IoT Core 2](/images-sitewise/iot-core-1.png)
* Create the Role

## Step 7 - Test Sending Data
Now that we have created our data ingest and connected it to SiteWise, we can test our data stream and see the graphs we created! Head over to device simulator and start our simulations for the devices we created.
* You can head over to the assets in SiteWise and under Measurements, the Latest Values should update
![Test](/images-sitewise/sitewise-test.png)
* Overtime, data will be populated and you can look at the resulting graphs!
![Final](/images-sitewise/sitewise-final.png)
* This is very useful for business purposes and gives an easy way to model industrial assets
* The time frame can be set along with zooming into graphs by selecting them

## Delete Resources
Congratulations, you’ve completed the SiteWise lab! Make sure to delete unnecessary resources to make sure that you do not incur a charge
* Delete the SiteWise Models, Assets, and Portals
* Delete/Stop the Device Simulator (keep it if you’re going to continue using it)
* Delete IoT Rules
