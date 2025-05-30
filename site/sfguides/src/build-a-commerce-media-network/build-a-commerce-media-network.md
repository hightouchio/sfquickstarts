author: Ian Maier
id: build_an_offsite_commerce_media_network_on_snowflake
summary: Build a commerce media platform for audience curation, offsite activation, and closed-loop measurement.
<!--- Categories below should be hyphenated, i.e., Getting-Started. Do not leave blank. Visit site for available categories. -->
categories: Marketing
environments: dais <!--- What is the correct environment for the DAIS hands-on lab? -->
status: Draft
feedback link: https://github.com/Snowflake-Labs/sfguides/issues
tags: Marketing, Partners, Media, Hightouch, Retail, Commerce

# Build an Offsite Commerce Media Network on Snowflake with Hightouch & The Trade Desk
<!-- ------------------------ -->
## Overview 
Duration: 1

In this guide you will learn how to transform your existing Snowflake investment into a user-friendly platform that a non-technical commerce media team can use to curate custom audiences, activate and share them with their advertising partners, and provide closed-loop attribution.

### Business Scenario
We are building a Travel Media Network for a global luxury hospitality conglomerate called Altuva, which owns multiple hotel brands with millions of  visitors annually. We are partnering with a high-end Consumer Packaged Goods company called Omnira, which sells luxury snacks, beverages, and health & wellness amenities within most of Altuva's hotels. Omnira has approached us with a specific campaign request: they want to promote their products to Altuva's hotel visitors in the 30 days before their arrival to influence more in-hotel consumption.

Our task is to build a platform that our non-techincal media team can use to create a highly customized audience, share the audience with Omnira, and provide Omnira with attribution reports to help them understand how well their ads drive in-hotel purchases.

### Prerequisites
- N/A

### What You’ll Learn 
- How to enrich Snowflake with UID2s using the Snowflake Native App for privacy-safe activation and measurement.
- How to build a visual audience builder on top of Snowflake using Hightouch.
- How to activate audiences on The Trade Desk using Hightouch, for both managed service and self-service campaign execution.
- How to send offline conversion events from Snowflake to The Trade Desk using Hightouch.
- How to execute The Trade Desk campaigns using shared audiences and conversion events.
- How to build SKU-level attribution reporting on Snowflake using The Trade Desk's Raw Event Data Stream (REDS). 

### What You’ll Need 
- A Snowflake account login with a role that has the permissions to create a new database, schema, and warehouse to be used by Hightouch. Account details will be provided to you for the live hands-on lab at Snowflake Summit.
- A Hightouch account with Customer Studio access. An account with temporary product access will be provided to you for the live hands-on lab at Snowflake Summit.

### What You’ll Build 
- A Snowflake-native visual audience builder customized to your luxury hospitality brand's unique data & offering.
- An activation platform for syndicating audiences and conversion events to The Trade Desk.
- Closed-loop attribution reporting with SKU-level breakdowns.

<!-- ------------------------ -->
## Enrich the Snowflake Customers table with UID2
Duration: 1

These instructions provide basic setup details for this hands-on lab. Implementation for your business may vary based on your unique data, use cases, and requirements. For detailed implementation instructions, visit the [UID2 Snowflake Integration Guide &rarr;](https://unifiedid.com/docs/guides/integration-snowflake)

The UID2 Snowflake Native App provides a simplified and seamless process for matching your customer identifiers (e.g. emails and phone numbers) to a pseudonymous identifier that can be used for ad targeting and conversion across publishers supported by The Trade Desk. Unlike directly identifying information, UID2 cannot by itself be traced back to the customer's original identity.

### How it works
The UID2 Snowflake Native App allows you to enrich your customer tables with UID2 data through authorized functions and views so that you can join UID2 data from the private tables to your customers' emails and phone numbers. You can’t access the private tables directly. The UID2 Share reveals only essential data needed to perform the UID2 enrichment tasks.

![UID2 Snowflake Native App Diagram](https://unifiedid.com/assets/images/uid2-snowflake-integration-architecture-drawio-e072ef23e1ed488eabe7e9e242c67cd2.png)

### Access the UID2 Share
To request access to the UID2 Share, complete the following steps:

1. Log in to the Snowflake Data Marketplace and select the UID2 listing:
    [Unified ID 2.0: Advertiser and Data Provider Identity Solution](https://app.snowflake.com/marketplace/listing/GZT0ZRYXTN8/unified-id-2-0-unified-id-2-0-advertiser-and-data-provider-identity-solution)
2. In the **Personalized Data** section, click **Request Data**.
3. Follow the onscreen instructions to verify and provide your contact details and other required information.
4. If you are an existing client of The Trade Desk, include your partner and advertiser IDs issued by The Trade Desk in the **Message** field of the data request form.
5. Submit the form.

After your request is received, a UID2 administrator will contact you with the appropriate access instructions. For details about managing data requests in Snowflake, see the [Snowflake documentation](https://docs.snowflake.com/en/user-guide/data-marketplace-consumer.html).

### Join UID data to your customer table

To map UID2 to emails and phone numbers in your customer table, use the `FN_T_IDENTITY_MAP` function.

```sql
select a.ID, a.EMAIL, m.UID, m.BUCKET_ID, m.UNMAPPED
from HOL_CMN.PUBLIC.CUSTOMERS a
LEFT JOIN(
  select ID, t.*
  from HOL_CMN.PUBLIC.CUSTOMERS, lateral UID2_PROD_UID_SH.UID.FN_T_IDENTITY_MAP(EMAIL, 'email') t) m
on a.ID=m.ID;
}
```

### Normalization

**Email Normalization**: The function accepts raw and hashed emails. If the identifier is an email address, the service normalizes the data using the [UID2 Email Address Normalization](https://unifiedid.com/docs/getting-started/gs-normalization-encoding#email-address-normalization) rules.

**Phone Normalization**: The function accepts raw and hashed phone numbers. If the identifier is a phone number, you must normalize it before sending it to the service, using the [UID2 Phone Number Normalization](https://unifiedid.com/docs/getting-started/gs-normalization-encoding#phone-number-normalization) rules.

<!-- ------------------------ -->
## Connect Snowflake to Hightouch
Duration: 1

Hightouch transforms Snowflake into a customer data platform (CDP), giving your commerce media team the ability to curate customer audiences, activate them for onsite and offsite targeting, and provide attribution and incrementality reporting without the need for engineering support. As the leading Snowflake-native CDP, Hightouch does not copy and store your data in its own infrastructure, ensuring that Snowflake remains your single source of truth and that your customer data remains private and secure. 

![Hightouch Snowflake CDP Diagram](https://quickstarts.snowflake.com/guide/hightouch_cdp/img/a9f317db6a84b731.png)

To connect Hightouch to Snowflake:
1. [Log in](https://app.hightouch.com/login) to your Hightouch account.
2. Navigate to **Integrations > Sources** and click **Add Source**.
  ![](assets/2_2_add_source.png)
3. Select **Snowflake** in the **Data systems** section.
  ![](assets/2_3_select_snowflake.png)
4. Provide the following configuration values in the **Configure your Snowflake source** section:
    - **Account identifier**: TBD
    - **Warehouse**: TBD
    - **Database**: TBD
  ![](assets/2_4_input_snowflake_config.png)
5. Scroll down and provide the following Snowflake credentials to authenticate and click **Continue**.
    - **Username**: TBD
    - **Role**: TBD
    - **Authentication method**: Password
    - **Password**: TBD
  ![](assets/2_5_provide_snowflake_credentials.png)
6. Name your data source "Snowflake - Altuva" and click **Finish**.

<!-- ------------------------ -->
## Connect Hightouch to The Trade Desk
Duration: 1

Next, we will connect Hightouch to The Trade Desk so that you can sync audiences and conversion events from Snowflake directly to Omnira's advertising account.

Hightouch also offers media networks the option to list their audiences on The Trade Desk's third-party data marketplace where their advertising partners can access and use them without a direct connection. Through this endpoint, the media network can set access permissions and rates. The Trade Desk will then track usage across their advertising partners, bill those partners for their usage, and pay the media network for that usage.

For the purpose of this guide, however, we will send audiences directly to the advertising partner.

### Create a Trade Desk destination
To push data to The Trade Desk, you need to create a destination.

Hightouch offers integrations with a number of different audience and conversion endpoints with The Trade Desk, including first-party audiences, CRM audiences, and third-party audiences that allow you to list them on the 3rd party data marketplace.

In this guide, we will connect to The Trade Desk using the first-party endpoints. Once the audiences and conversion events are synced to The Trade Desk, we will have to option of executing campaigns from our own advertising account (e.g., for managed services) or share the audience with our advertising partner's account (e.g., for self-service).

To connect The Trade Desk destination:
1. Navigate to **Integrations > Destinations** and click **Add Destination**.
    ![](assets/1_1_create_destination.png)
3. Search for **The Trade Desk**, select it, and click **Continue**.
    ![](assets/1_2_choose_ttd_destination.png)
4. Scroll down to the **First-Party Data Segment** section. Input the following values in the config:
    - **Environment**: US West
    - **Advertiser ID**:  `rz1pddq`
    - **Advertiser secret key**: `jb1t6wx8dffx71ydyephpv5k121oxi28`
    ![](assets/1_3_1p_segment_credentials.png)
5. Scroll down further to the **Conversion event integration** section. Input the following values in the config for Omnira:
    - **Advertiser ID**: `z2jxhwl`
    - **Advertiser secret key**: `z2jxhwl`
    - **Offline data provider ID**
    - **Offline tracking tag API token**
    ![](assets/1_4_ttd_offline_conversion_credentials.png)
6. Name the destination "The Trade Desk - Omnira" and click **Finish** to create the destination.

<!-- ------------------------ -->
## Build the audience schema
Duration: 1

Data schemas define how data is structured, including the data types (like users, events, accounts, and households) and relationships between those data types. If you think of your data warehouse like an actual house, your data schema is the blueprint. Just like a blueprint outlines the structure of a house—where the rooms are and how they connect—a data schema defines how your data is organized and how different parts of your data relate to one another.

For our Travel Media Network at Altuva, there are three data types we need to represent in our schema using a Snowflake table:
1. **Customers** - Information about our customers, including their identity and loyalty membership status.
2. **Hotel Visits** - Information about past, present, and future hotel visits, including when and where our customers are visiting our hotels.
3. **Purchase Events** - Past and present amenities purchased in our hotels.

![Altuva Travel Media Network Schema](assets/5_6_purchases_parent_model_end.png)

### Create a Customer parent model
Parent models define the primary dataset you want to build your audiences off. In this guide, we will create a Customers table that represents all Altuva Brands customers who have visited a hotel and/or signed up as a loyalty member.

To create a [parent model](https://hightouch.com/docs/customer-studio/schema#parent-model-setup) for Altuva use the following steps:
1. Create a parent model
    ![](assets/3_1start_schema_setup.png)
    1. Go to the [Schema page](https://app.hightouch.com/schema-v2/view).
    2. Select your Snowflake data source from Step 5 by selecting it from the dropdown.
    3. Click **Create parent model**.
2. Select the **CUSTOMERS** table, preview the results, and click **Continue**.
    ![](assets/3_2select_customer_table.png)
3. Configure the Customers model:
    ![](assets/3_6_select_customer_table.png)
    1. Enter a **name** for your parent model. In this example, use “Customers”.
    2. Select a **primary key**. You must select a column with unique values. In this example, use **CUSTOMER_ID** for the primary key.
    3. Select columns for the model's **primary label** and **secondary label**. Hightouch displays these column values when previewing audiences. In this example, we will use **FIRST_NAME** as the primary label and **LAST_NAME** as the secondary label.
    4. Click **Create parent model**.

The parent model now appears in the schema builder. You can edit an existing parent model by clicking on it and making changes in the Query tab. For more configuration options, refer to the [column configuration](https://hightouch.com/docs/customer-studio/schema#column-configuration) section.

![](assets/3_10_customers_model_end.png)

### Create Hotel Visits related model
Related models and events provide additional characteristics or actions on which you can filter your parent model. When you create a related model, you simultaneously set up the relationship between it and other objects in the schema.

For our Travel Media Network we will create a related model for Hotel Visits so that we can build audiences based on past or upcoming visits to certain hotels. This model will have a 1:Many relationship between Customers and Hotel Visits since an individual customer can make multiple hotel visits.

1. Click the **+** button on the **Customers parent model** that you created before and select **Create a related model**.
    ![](assets/4_1_create_related_model.png)
2. Select the **HOTEL_VISITS** table using the table selector, **preview the results**, and click **Continue**.
    ![](assets/4_2_select_hotel_visits_table.png)
3. Configure the model:
    ![](assets/4_3_configure_hotel_visits_model.png)
    1. Enter a **name** for your related model. In this example, name it “Hotel Visits”.
    2. Define the related model's **Relationship**. In this example, select the relationship's cardinality as **1:Many**.
    3. To join rows from the related model to rows in the parent model, select the relevant **foreign key columns** from the parent (Customers) and related model (Hotel Visits). In this example, select **Customer_ID** as the foreign key for both models.
    4. Click **Create related model**.

The related model now appears in the schema builder. You can edit an existing related model by clicking on it and making changes in the Query tab.

![](assets/4_4_hotel_visits_model_end.png)

### Create a Purchases parent model
In order to support each advertising partner with closed-loop measurement, we will want to be able to use the audience builder to quickly create filtered-down conversion event feeds. In this example, we want to make sure that we are only sending Omnira Purchase event data from products that they sell (and that we are not sending purchase events for competing products).

To do this, we will create a second Parent model for **Purchases**. These parent model will be linked to the Customers parent model that we can do two things with the data:
1. Exclude customers from our audiences if they purchased an Omnira product recently
2. Send Purchase conversion events to Omnira's advertising account when the purchase includes an Omnira product

To create and link two parent models together:
1. Click the **Create** button at the top right of the Schema builder page and select **Parent model**.
    ![](assets/5_1_create_parent_model_purchases.png)
2. Select the **PURCHASES** table using the table selector, preview the results, and click **Continue**.
    ![](assets/5_2_select_purchases_table.png)
3. Configure the events model:
    ![](assets/5_3_configure_purchases_parent_model.png)
    1. Enter a **name** for your event model. For this example, use "Purchases".
    2. Set the appropriate **Primary Key** field to determine how to distinguish between events. In this example, select **PURCHASE_ID**. Set the **Primary key** to **PURCHASE_ID** and the **Secondary Label** to **PRODUCT_NAME**. Click **Create parent model**.
    ![](assets/5_4_add_purchases_relationship.png)
    7. You should now see a new Parent model in the schema builder. To link the Purchases and Customers tables, click on the **Purchases** parent model and click **Relationships** > **Add Relationship**.
    ![](assets/5_5_configure_purchases_relationship.png)
8.  Use the selectors to create a **many:1** relationship between Purchases and Customers. Set the foreign keys so that both tables are linked using the **CUSTOMER_ID** field in each respective model.
    9. Click **Save changes**.

The **Purchases** parent model will now be linked to the Customers parent model. Now you will be able to use Hightouch's visual audience builder to create audiences for targeting and filtered purchase events for conversion tracking.

![](assets/5_6_purchases_parent_model_end.png)

<!-- ------------------------ -->
## Create a custom audience on Snowflake
Duration: 1

Now that we've built the schema for audience building and conversion tracking, we can use Hightouch's audience builder to create a custom audience for our advertising partner's upcoming campaign.

Omnira wants to run an influence campaign to encourage Altuva visitors to buy more of their products when they visit Altuva hotels. But they want to carefully segment the audience to ensure the campaign is as effective as possible:
- Target customers with a visit coming up in the next 30 days
- Target members with Platinum or Diamond loyalty tiers to promote a discount offer 
- Exclude visitors of the Cavara Residences, since they don’t offer their products at that hotel brand
- Exclude recent purchasers

For most CDPs and audience managers, this audience would be too complex to build without the help of data engineering since the platforms don't support concepts like "hotel visits". But with the customized schema we built using our Snowflake data, a business user in Hightouch can build this audience in a few minutes without technical limitations.

### Build a custom audience on Snowflake
To build the audience:
1. Go to **Customer Studio > Audience** and click **Add audience**.
    ![](assets/6_1_add_audience.png)
2. Select the **Customers** parent model.
    ![](assets/6_2_select_customers_parent_model.png)
3. Build the audience to match the definition here:
    ![](assets/6_0_audience_end.png)
    1. Select **Add filter** and choose **Customers > MEMBERSHIP_LEVEL**.
    ![](assets/6_3_select_membership_level_attribute.png)
    2. Select **Platinum** and **Diamond**.
    ![](assets/6_4_select_membership_level_values.png)
    3. Select **Add filter** and choose **Relations > Hotel Visits**.
    ![](assets/6_5_select_hotel_visits_relation.png)
    4. Select **Where event property is** and choose the **CHECK_IN_DATE** field, and select **within next 1 months**.
    ![](assets/6_6_define_checkin_filter.png)
    5. Select **Where event property is** again and choose the **HOTEL_BRAND** field. Click the **equals** box and change the logic to **does not equal** **Cavara Residences**.
    ![](assets/6_7_define_hotel_brand_filter.png)
    6. Select **Add filter** and choose **Events > Purchases**.
    ![](assets/6_8_select_purchases_filter.png)
    7. Click on the filter that says **at least 1 time** and change the frequency to **at most 0 times**.
    ![](assets/6_9_define_purchases_exclusion_filter.png)
    8. Select the **Time window filter** on the **Purchase event** and change the filter to **within the previous 1 year**.
    ![](assets/6_10_define_purchases_time_window.png)
    9. Scroll back to the top and click **Calculate size** to get an estimate of the number of customers in the audience. Click **Continue** at the bottom.
    ![](assets/6_11_calculate_audience_size.png)
    10.  Name the audience “Upcoming Visitors to Altuva Hotels” and click **Finish** to save the audience.

<!-- ------------------------ -->
## Sync the custom audience to The Trade Desk
Duration: 1

Now we will sync the audience that we created to The Trade Desk for targeting. To enable self-service, we will share the audience with Omnira's seat.

### Configure The Trade Desk first-party data segment sync
1. Click **Add sync** from the Upcoming Visitors to Altuva Hotels audience page.
    ![](assets/9_1_add_sync_v2.png)
2. Select **The Trade Desk - Omnira**.
    ![](assets/9_2_select_ttd_sync_destination_v2.png)
3. Select **First-party data segment**. This will allow you to create, sync, and refresh a first-party audience to The Trade Desk using UID2 as the match key.
    ![](assets/7_3_select_1p_data_segment.png)
4. Keep the **create new segment** setting on and leave the **segment name** blank. The sync will automatically inherit the audience name, “Upcoming Visitors - Omnira”.
    ![](assets/7_4_new_segment_name_blank.png)
5. Select the **UID2** field from your Snowflake Customers table. Make sure it is mapped to the **Unified ID 2.0 (UID2)** field from The Trade Desk. Click **Continue**.
    ![](assets/7_5_map_uid2.png)
6. Set the **Schedule Type** to **Manual** and click **Finish**. In a live environment, we would set the schedule on an interval to refresh daily so that Hightouch automatically adds and removes the appropriate users daily, ensuring your audience is always fresh and complies with opt-out requests.
    ![](assets/7_6_set_sync_schedule.png)

### Run and observe the sync
1. To run the sync manually, click **Run Sync**.
    ![](assets/7_7_run_sync.png)
2. To observe the status of the sync run, click on the **processing** sync under **Recent sync runs**.
    ![](assets/7_8_select_run.png)

The Run Summary page provides granular insight into the status and speed of the data being queried on Snowflake and synced to The Trade Desk. This level of observability helps your team easily monitor, diagnose, and fix any issues with slow or failed audience updates. You also have the option to set up alerting so that your team can proactively identify and fix issues.
![](assets/7_8_review_sync_run.png)

<!-- ------------------------ -->
## Advertiser: Run a self-service campaign
Duration: 1

Now that Omnira has access to the audience, they can select and target that audience in their campaigns by adding the audience to an ad group.

![](assets/8_1_edit_audience.png)

They can also preview the audience before using it to see the number of people, households, and devices that they can reach.

![](assets/8_2_preview_audience.png)

<!-- ------------------------ -->
## Create a brand-specific conversion feed
Duration: 1

Now that Omnira is actively targeting hotel visits with targeted ads, they want to track the success of their campaign in The Trade Desk by syncing conversion events to their advertising seat. However, the purchases happen through Altuva meaning that they do not have direct access to conversion data.

In order to get access to conversions, we will need to create and share conversion data with Omnira in a way where only Omnira purchases are shared and Altuva's customer data is kept private and secure.

1. Navigate to **Customer Studio** > **Audiences** in Hightouch and click **Add audience**.
    ![](assets/)
2. Select the **Purchases** parent model so that we can build an audience of purchase events.
    ![](assets/)
3. Click **Add filter** in the audience builder and select the **PURCHASE_DATE** property so that we can filter to recent conversion events.
    ![](assets/)
4. Set the **PURCHASE_DATE** time window to **within previous 1 week**.
    ![](assets/)
5. Click **Add filter** in the audience builder and select the **BRAND** property. Add a **contains Omnira** filter to ensure only Omnira purchase data is shared. Click **Continue**.
    ![](assets/)
6. Name the audience "Omnira Purchases" and click **Finish**.

Now you have a brand-specific conversion audience that you can sync to The Trade Desk. You can easily clone this audience to create new conversion feeds for other brands.

<!-- ------------------------ -->
## Sync the brand-specific conversion data to The Trade Desk
Duration: 1

To share the brand-specific Purchase events with Omnira, we will send the conversion data directly to their advertising seat so that they can use it for optimization and reporting:
1. Click **Add Sync** from the Omnira Purchases audience page.
    ![](assets/)
2. Select the The Trade Desk - Omnira destination.
    ![](assets/)
3. Select the **Offline conversion events** configuration option.
    ![](assets/)
4. Keep the default tracking tag configuration options set and name the offline tracking tag "Altuva Purchases".
    ![](assets/)
5. Select **Purchases** for the event name set up the match values so that The Trade Desk can match conversions back to specific impressions and clicks for attribution. Select the **UID2** field from the **Omnira Purchases** model and map it to the **UID2** value under the **destination field**.
    ![](assets/)
6. Configure the conversion event metadata and click **Continue**.
    ![](assets/)
    1. **PURCHASE_DATE** -> **TimestampUtc**
    2. **TOTAL_PRICE** -> **Value**
    3. **CURRENCY** -> **ValueCurrency**
    4. **Merchant_ID** -> **MerchantId**
8. Define the sync schedule. In this example, set the schedule on an **Interval**. Set the interval to **every 1 day(s)**. Click **Finish**.

You have successfully set up a conversion feed that will send new conversions to Omnira's advertising account, using UID2 to privately and securely match conversions to impressions and clicks from their ad campaigns.

<!-- ------------------------ -->
## Use conversion data for measurement & reporting
Duration: 1

Now that the conversion data has been shared with Omnira, they can use that data for campaign measurement and reporting in their self-service campaign.

Conversion events can be found within The Trade Desk by navigating to the **Advertiser Data & Identity** tile.

![](assets/)

<!-- ------------------------ -->
## (Bonus) Set up REDS data sync
Duration: 1

<!-- ------------------------ -->
## (Bonus) Create an attribution report
Duration: 1

<!-- ------------------------ -->
## Conclusion
Duration: 0





<!-- ------------------------ -->
## Code Snippets, Info Boxes, and Tables
Duration: 2

Look at the [markdown source for this sfguide](https://raw.githubusercontent.com/Snowflake-Labs/sfguides/master/site/sfguides/sample.md) to see how to use markdown to generate code snippets, info boxes, and download buttons. 

### JavaScript
```javascript
{ 
  key1: "string", 
  key2: integer,
  key3: "string"
}
```

### Java
```java
for (statement 1; statement 2; statement 3) {
  // code block to be executed
}
```

### Info Boxes
> aside positive
> 
>  This will appear in a positive info box.


> aside negative
> 
>  This will appear in a negative info box.

### Buttons
<button>

  [This is a download button](link.com)
</button>

### Tables
<table>
    <thead>
        <tr>
            <th colspan="2"> **The table header** </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>The table body</td>
            <td>with two columns</td>
        </tr>
    </tbody>
</table>

### Hyperlinking
[Youtube - Halsey Playlists](https://www.youtube.com/user/iamhalsey/playlists)

<!-- ------------------------ -->
## Images, Videos, and Surveys, and iFrames
Duration: 2

Look at the [markdown source for this guide](https://raw.githubusercontent.com/Snowflake-Labs/sfguides/master/site/sfguides/sample.md) to see how to use markdown to generate these elements. 

### Images
![Puppy](assets/SAMPLE.jpg)

### Videos
Videos from youtube can be directly embedded:
<video id="KmeiFXrZucE"></video>

### Inline Surveys
<form>
  <name>How do you rate yourself as a user of Snowflake?</name>
  <input type="radio" value="Beginner">
  <input type="radio" value="Intermediate">
  <input type="radio" value="Advanced">
</form>

### Embed an iframe
![https://codepen.io/MarioD/embed/Prgeja](https://en.wikipedia.org/wiki/File:Example.jpg "Try Me Publisher")

<!-- ------------------------ -->
## Conclusion
Duration: 1

At the end of your Snowflake Guide, always have a clear call to action (CTA). This CTA could be a link to the docs pages, links to videos on youtube, a GitHub repo link, etc. 

If you want to learn more about Snowflake Guide formatting, checkout the official documentation here: [Formatting Guide](https://github.com/googlecodelabs/tools/blob/master/FORMAT-GUIDE.md)

### What we've covered
- creating steps and setting duration
- adding code snippets
- embedding images, videos, and surveys
- importing other markdown files
