Follow the steps below to add the PriceList module to your project.

##Changes in `pom.xml` files 
###In you project POM Declare the the BLC snapshot repository

```xml
	<repositories>
		<repository>
			<id>public releases</id>
			<name>public releases</name>
			<url>http://www.broadleafcommerce.org/nexus/content/repositories/snapshots/</url>
		</repository>
	</repositories>
```
	
###In you Project `pom.xml` add the dependency for the Broadleaf PriceList module

```xml
<dependency>
	<groupId>org.broadleafcommerce</groupId>
	<artifactId>broadleaf-pricelist</artifactId>
	<version>1.0.0-SNAPSHOT</version>
	<type>jar</type>
	<scope>compile</scope>
</dependency>
```

###In your SITE `pom.xml` where you will extend  Customer, Offer, Order, ProductOptionValue, SearchFacetRange, SkuBundleItem and Sku

```xml
<dependency>
	<groupId>org.broadleafcommerce</groupId>
	<artifactId>broadleaf-pricelist</artifactId>
</dependency>
```

###In your ADMIN `pom.xml`

```xml
<dependency>
	<groupId>org.broadleafcommerce</groupId>
	<artifactId>broadleaf-pricelist</artifactId>
</dependency>
```

##Changes in `web.xml`

###Add `classpath:/bl-pricelist-applicationContext.xml` and  `classpath:/bl-pricelist-admin-applicationContext.xml`in the `<context-param />` section

```xml
<context-param>
	<param-name>patchConfigLocation</param-name>
	<param-value>
	.
	classpath:/bl-pricelist-admin-applicationContext.xml
	classpath:/bl-pricelist-applicationContext.xml
	.
	.
	</param-value>
</context-param>
```
##Changes in `applicationContext-filter.xml` and `applicationContext-filter-combined.xml` 

###Add `priceListDynamicSkuPricingFilter` in the   `<sec:filter-chain pattern= />` section

```
   <sec:filter-chain pattern="/**" filters=""
               blURLHandlerFilter,
               priceListDynamicSkuPricingFilter"/>
```


##Changes in `mycompanyAdmin.gwt.xml`

###Add the following line

```xml
<inherits name="org.broadleafcommerce.admin.priceListModule" />
```

##Domain Changes

###Extend Customer, Offer, Order, ProductOptionValue, SearchFacetRange, SkuBundleItem and Sku.
Be sure you are comfortable with [[extending entities | Extending Entities Tutorial]] before continuing on.

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@Table(name = "BLC_PRICE_ADJUSTMENT")
public class MyProductOptionValueImpl extends ProductOptionValueImpl implements PriceListProductOptionValue {…}
```


###Embed the PriceAdjustment objects that you want to add

```java
@Embedded
protected PriceListProductOptionValueImpl embeddablePriceList = new PriceListProductOptionValueImpl();

```

###Implement Delegate Methods 
Your IDE should have the functionality to implement delegate methods (Generate code > Delegate Methods). 

We will need to take a few addiational steps to make sure that our embeddable object is always present. Due to how hibernate handles empty embeddables if no data is inserted into the database the embeddable object will remain null. To address this issue we will need to implement a lazy initialization of the embeddable objects. 

```java
protected void initializePriceListProductOptionValue(){
	if(embeddablePriceList == null){
		embeddablePriceList = new PriceListProductOptionValueImpl();
	}
}
```

Include the initialization method in all delegate methods.

```java
@Override
@Nullable
public String getAttributeValue() {
	initializePriceListProductOptionValue();
	return embeddablePriceList.getAttributeValue();
}
```

Add the necessary module keys in the database

```sql
INSERT INTO BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE) VALUES (64,'Create PriceList','PERMISSION_CREATE_PRICELIST', 'CREATE');
INSERT INTO BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE) VALUES (65,'Update PriceList','PERMISSION_UPDATE_PRICELIST', 'UPDATE');
INSERT INTO BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE) VALUES (66,'Delete PriceList','PERMISSION_DELETE_PRICELIST', 'DELETE');
INSERT INTO BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE) VALUES (67,'Read PriceList','PERMISSION_READ_PRICELIST', 'READ');
INSERT INTO BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE) VALUES (68,'All PriceList','PERMISSION_ALL_PRICELIST', 'ALL');
INSERT INTO BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE) VALUES (79,'Create PriceList Rules','PERMISSION_CREATE_PRICELISTRULE', 'CREATE');
INSERT INTO BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE) VALUES (80,'Update PriceList Rules','PERMISSION_UPDATE_PRICELISTRULE', 'UPDATE');
INSERT INTO BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE) VALUES (81,'Delete PriceList Rules','PERMISSION_DELETE_PRICELISTRULE', 'DELETE');
INSERT INTO BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE) VALUES (82,'Read PriceList Rules','PERMISSION_READ_PRICELISTRULE', 'READ');
INSERT INTO BLC_ADMIN_PERMISSION (ADMIN_PERMISSION_ID, DESCRIPTION, NAME, PERMISSION_TYPE) VALUES (83,'All PriceList Rules','PERMISSION_ALL_PRICELISTRULE', 'ALL');


INSERT INTO BLC_ADMIN_ROLE_PERMISSION_XREF (ADMIN_ROLE_ID, ADMIN_PERMISSION_ID) VALUES (1,83);
INSERT INTO BLC_ADMIN_ROLE_PERMISSION_XREF (ADMIN_ROLE_ID, ADMIN_PERMISSION_ID) VALUES (2,83);
INSERT INTO BLC_ADMIN_SECTION (ADMIN_SECTION_ID, ADMIN_MODULE_ID, NAME, SECTION_KEY, URL, USE_DEFAULT_HANDLER) VALUES (26, 1, 'Price List', 'PriceList', '/pricelist', TRUE);
INSERT INTO BLC_ADMIN_SECTION (ADMIN_SECTION_ID, ADMIN_MODULE_ID, NAME, SECTION_KEY, URL, USE_DEFAULT_HANDLER) VALUES (27, 1, 'Price List Rule', 'PriceListRule', '/pricelist-rule', TRUE);
INSERT INTO BLC_ADMIN_SECTION_PERMISSION_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (26,64);
INSERT INTO BLC_ADMIN_SECTION_PERMISSION_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (26,65);
INSERT INTO BLC_ADMIN_SECTION_PERMISSION_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (26,66);
INSERT INTO BLC_ADMIN_SECTION_PERMISSION_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (26,67);
INSERT INTO BLC_ADMIN_SECTION_PERMISSION_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (26,68);
INSERT INTO BLC_ADMIN_SECTION_PERMISSION_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (27,79);
INSERT INTO BLC_ADMIN_SECTION_PERMISSION_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (27,80);
INSERT INTO BLC_ADMIN_SECTION_PERMISSION_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (27,81);
INSERT INTO BLC_ADMIN_SECTION_PERMISSION_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (27,82);
INSERT INTO BLC_ADMIN_SECTION_PERMISSION_XREF (ADMIN_SECTION_ID, ADMIN_PERMISSION_ID) VALUES (27,83);
 	 
 ```