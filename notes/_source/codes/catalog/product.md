# 产品 Product

## 获取产品集合

```php
/* @var $objectManager \Magento\Framework\ObjectManagerInterface */
$collection = $objectManager->get( \Magento\Catalog\Model\ProductFactory::class )->create()->getCollection();
```

以下为 AND 条件使用实例：
```php
$collection
    ->addAttributeToFilter( 'news_from_date', [ 'to' => date( 'Y-m-d' ) ] )
    ->addAttributeToFilter( 'title', [ 'like' => '%abc' ] );
```

以下为 OR 条件使用实例：
```php
$collection->addAttributeToFilter( [
    [ 'attribute' => 'news_from_date', 'from' => date( 'Y-m-d' ) ],
    [ 'attribute' => 'news_to_date', 'to' => date( 'Y-m-d' ) ]
] );
```

下面列出可用条件：

```
'eq'            => "{{fieldName}} = ?",
'neq'           => "{{fieldName}} != ?",
'like'          => "{{fieldName}} LIKE ?",
'nlike'         => "{{fieldName}} NOT LIKE ?",
'in'            => "{{fieldName}} IN(?)",
'nin'           => "{{fieldName}} NOT IN(?)",
'is'            => "{{fieldName}} IS ?",
'notnull'       => "{{fieldName}} IS NOT NULL",
'null'          => "{{fieldName}} IS NULL",
'gt'            => "{{fieldName}} > ?",
'lt'            => "{{fieldName}} < ?",
'gteq'          => "{{fieldName}} >= ?",
'lteq'          => "{{fieldName}} <= ?",
'finset'        => "FIND_IN_SET(?, {{fieldName}})",
'regexp'        => "{{fieldName}} REGEXP ?",
'from'          => "{{fieldName}} >= ?",
'to'            => "{{fieldName}} <= ?",
'seq'           => null,
'sneq'          => null,
'ntoa'          => "INET_NTOA({{fieldName}}) LIKE ?",
```

### 为集合的产品添加访问次数属性

```php
/* @var $objectManager \Magento\Framework\ObjectManagerInterface */
$from = date( 'Y-m-d', time() - 3600 * 24 * 30 );
$to = date( 'Y-m-d' );
$collection = $objectManager->create( \Magento\Reports\Model\ResourceModel\Product\Collection::class );
$collection->addViewsCount( $from, $to ); // $from 及 $to 为可选参数
foreach( $collection as $product ) {
    $views = $product->getData( 'views' );
}
```

p.s. 开启 full page cache 可能使该方法的结果变得非常不准确，需设置关闭产品详细页缓存。


### 获取多个指定分类的产品

可选用两个可选条件 ```in``` 或 ```nin```。

```php
/* @var $objectManager \Magento\Framework\ObjectManagerInterface */
/* @var $categoryIds array */
$collection = $objectManager->create( \Magento\Catalog\Model\ResourceModel\Product\Collection::class )
        ->addCategoriesFilter( [ 'in' => $categoryIds ] );
```


### 获取可见产品集合

如果执行 load 方法则无需添加 stock 相关代码，系统原生插件在 load 之前添加了库存过滤。

```php
/* @var $objectManager \Magento\Framework\ObjectManagerInterface */
/* @var $catalogProductVisibility \Magento\Catalog\Model\Product\Visibility */
$productCollection = $objectManager->create( \Magento\Catalog\Model\ResourceModel\Product\Collection::class );
//$productCollection->setVisibility( $catalogProductVisibility->getVisibleInSearchIds() );
//$productCollection->setVisibility( $catalogProductVisibility->getVisibleInCatalogIds() );
$productCollection->setVisibility( $catalogProductVisibility->getVisibleInSiteIds() );

/* @var $stockConfig \Magento\CatalogInventory\Model\Configuration */
/* @var $stockHelper \Magento\CatalogInventory\Helper\Stock */
if ( !$stockConfig->isShowOutOfStock() ) {
    $stockHelper->addInStockFilterToCollection( $productCollection );
}
```


### 为产品集合加载多个 store 的属性数据

```php
/* @var $productCollection \Magento\Catalog\Model\ResourceModel\Product\Collection */
/* @var $alias string */
/* @var $attribute string */
/* @var $bind string */
/* @var $filter string */
/* @var $joinType string */
/* @var $storeId int */
$productCollection->joinAttribute( $alias, $attribute, $bind, $filter, $joinType, $storeId );
```

- ***$alias*** - 别名，使用 getData 方法时用到的名字
- ***$attribute*** - 属性名，格式为 `[entity_type]/[attribute_code]`，比如 `catalog_product/name`
- ***$bind*** - 指定用作关联的产品 attribute code
- ***$filter*** - 指定的产品数据表中用作关联的字段，默认为 `row_id`
- ***$joinType*** - inner / left
- ***$storeId*** - Store View ID


### 联合其他数据表查询

```php
/* @var $productCollection \Magento\Catalog\Model\ResourceModel\Product\Collection */
/* @var $table string|array */
/* @var $bind string */
/* @var $fields array */
/* @var $cond null|string|array */
/* @var $joinType null|string */
$productCollection->joinTable( $table, $bind, $fields, $cond, $joinType );
```

- ***$table*** - 联合的数据表，必须
- ***$bind*** - 联合条件，格式为 `field_name=attribute_code`，必须
- ***$fields*** - 联合的表中要查询的字段，格式为 `[ field_alias => field_name ]`，必须
- ***$cond*** - 额外的联合条件。
以字符串作参数时，系统会将 `{{table}}` 解释为联合数据表的别名，比如 `{{table}}.id = 123`；
以数组作参数时，格式为 `[ field_name => [ condition => value ] ]`，其中 `field_name` 是联合数据表当中的字段名
- ***$joinType*** - inner / left


### 同时获取多个 Website 的产品页链接

```php
/* @var $storeManager \Magento\Store\Model\StoreManagerInterface */
/* @var $productCollectionFactory \Magento\Catalog\Model\ResourceModel\Product\CollectionFactory */
foreach ( $storeManager->getWebsites() as $website ) {
    $storeManager->setCurrentStore( $website->getDefaultStore() );

    $productCollection = $productCollectionFactory->create()
            ->setStore( $website->getDefaultStore()->getId() )
            ->addWebsiteFilter( $website->getId() )
            ->load()
            // Below methods only run completely after collection loaded
            ->addUrlRewrite();

    foreach ( $productCollection as $product ) {
        $product->getProductUrl();
    }
}
```



## 产品属性


### 获取属性的所有可选项

如果已经指定产品，可以
```php
/* @var $product \Magento\Catalog\Model\Product */
/* @var $attributeCode string */
/* @var $withEmptyOption boolean */
$options = $product->getResource()->getAttribute( $attributeCode )
    ->getSource()->getAllOptions( $withEmptyOption );
```

或者
```
/* @var $objectManager \Magento\Framework\ObjectManagerInterface */
/* @var $attributeCode string */
/* @var $withEmptyOption boolean */
$objectManager->create( \Magento\Catalog\Model\Entity\Attribute::class )
    ->loadByCode( \Magento\Catalog\Api\ProductAttributeInterface::ENTITY_TYPE_CODE, $attributeCode )
    ->getSource()->getAllOptions( $withEmptyOption );
```


### 获取属性值的 Label

如果已经指定产品，可以
```php
/* @var $product \Magento\Catalog\Model\Product */
/* @var $attributeCode string */
$valueTxt = $product->getAttributeText( $attributeCode );
```

或者通过属性 code 或 ID 及值的 ID 获取值名
```php
/* @var $objectManager \Magento\Framework\ObjectManagerInterface */
/* @var $attributeCode string */
/* @var $attributeOptionId int */
$objectManager->create( \Magento\Catalog\Model\Entity\Attribute::class )
    ->loadByCode( \Magento\Catalog\Api\ProductAttributeInterface::ENTITY_TYPE_CODE, $attributeCode )
    ->getSource()->getOptionText( $attributeOptionId );
```


### 通过属性选项的 Admin Label 取值

```
/* @var $objectManager \Magento\Framework\ObjectManagerInterface */
/* @var $attributeCode string */
$options = $objectManager->create( \Magento\Catalog\Model\Entity\Attribute::class )
    ->loadByCode( \Magento\Catalog\Api\ProductAttributeInterface::ENTITY_TYPE_CODE, $attributeCode )
    ->getSource()->getAllOptions( false, true );

/* @var $label string */
foreach ( $options as $option ) {
    if ( $option['label'] == $label ) {
        return $option['value'];
    }
}
```

### 获取指定产品的所有属性

获得一个以属性 code 为键名的数组
```php
/* @var $productResource \Magento\Catalog\Model\ResourceModel\Product */
/* @var $attributeSetId int */
$attributes = $productResource->loadAllAttributes()->getAttributesByCode();
```

获取指定属性集的所有属性
```php
/* @var $productResource \Magento\Catalog\Model\ResourceModel\Product */
/* @var $attributeSetId int */
$attributes = $productResource->loadAllAttributes()->getSortedAttributes( $attributeSetId );
```


### 通过 Setup 添加产品属性

```
namespace Namespace\Module\Setup;

use Magento\Catalog\Model\Product;
use Magento\Eav\Model\Entity\Attribute\ScopedAttributeInterface;
use Magento\Eav\Setup\EavSetupFactory;
use Magento\Framework\DB\Ddl\Table;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\ModuleDataSetupInterface;

class InstallData implements \Magento\Framework\Setup\InstallDataInterface {

    /**
     * @var int
     */
    private $defaultProductAttributeSetId;

    /**
     * @var \Magento\Eav\Setup\EavSetup
     */
    private $eavSetup;

    /**
     * @var \Magento\Eav\Setup\EavSetupFactory
     */
    private $eavSetupFactory;

    public function __construct( EavSetupFactory $eavSetupFactory )
    {
        $this->eavSetupFactory = $eavSetupFactory;
    }

    private function addProductAttribute()
    {
        /* @var $attributeCode string */
        /* @var $attributeGroupCode string */

        $this->eavSetup->addAttribute( Product::ENTITY, $attributeCode, [
            'type' => 'int', // datetime, decimal, int, text, varchar
            'label' => 'Attribute Name',
            'input' => 'select', // select, text, date, hidden, boolean, multiline, textarea, image, media_image, multiselect
            'source' => 'Magento\Eav\Model\Entity\Attribute\Source\Table',
            'required' => false,
            'sort_order' => 602,
            'global' => ScopedAttributeInterface::SCOPE_GLOBAL,
            'user_defined' => false, // eav_attribute
            'is_visible_in_grid' => false,
            'is_filterable_in_grid' => false,
            'visible' => true, // whether to show in back-end
            'visible_on_front' => false, // whether to show in front-end product detail page
            'used_in_product_listing' => false, // whether to create in flat table
            'used_for_sort_by' => false // whether to add as an option of sort by in front-end
         ] );

        $groupId = (int) $this->eavSetup->getAttributeGroupByCode( Product::ENTITY, $this->defaultProductAttributeSetId, $attributeGroupCode, 'attribute_group_id' );
        $this->eavSetup->addAttributeToGroup( Product::ENTITY, $this->defaultProductAttributeSetId, $groupId, $attributeCode );
    }

    public function install( ModuleDataSetupInterface $setup, ModuleContextInterface $context )
    {
        $this->eavSetup = $this->eavSetupFactory->create( [ 'setup' => $setup ] );
        $this->defaultProductAttributeSetId = $this->eavSetup->getDefaultAttributeSetId( Product::ENTITY );

        $this->addProductAttribute();
    }
}
```


## 其他

### 创建一个新产品

以下方法没有使用 Product Repository 以便跳过必填属性检查：

```php
use Magento\Catalog\Model\Product\Attribute\Source\Status;
use Magento\Catalog\Model\Product\Type;
use Magento\Catalog\Model\Product\Visibility;
use Magento\ConfigurableProduct\Model\Product\Type\Configurable;
use Magento\Store\Model\Store;

/* @var $resourceModel \Magento\Catalog\Model\ResourceModel\Product */
/* @var $productFactory \Magento\Catalog\Model\ProductFactory */
/* @var $attributeSetId int */
/* @var $websiteIds array */
/* @var $categoryIds array */
/* @var $urlKey string */

$simpleProduct = $productFactory->create()->setStoreId( Store::DEFAULT_STORE_ID )
    ->setTypeId( Type::TYPE_SIMPLE )
    ->setAttributeSetId( $attributeSetId )
    ->setWebsiteIds( $websiteIds )
    ->setStatus( Status::STATUS_ENABLED )
    ->setVisibility( Visibility::VISIBILITY_NOT_VISIBLE )
    ->setCategoryIds( $categoryIds )
    ->setUrlKey( $urlKey );

$resourceModel->save( $simpleProduct );


/* @var $configAttribute \Magento\Catalog\Model\Entity\Attribute */
/* @var $optionsFactory \Magento\ConfigurableProduct\Helper\Product\Options\Factory */
/* @var $childrenIds array */

$configProduct = $productFactory->create()->setStoreId( Store::DEFAULT_STORE_ID )
    ->setTypeId( Configurable::TYPE_CODE )
    ->setAttributeSetId( $attributeSetId )
    ->setWebsiteIds( $websiteIds )
    ->setStatus( Status::STATUS_ENABLED )
    ->setVisibility( Visibility::VISIBILITY_BOTH )
    ->setCategoryIds( $categoryIds )
    ->setUrlKey( $urlKey );

$attributeValues = [];
foreach ( $configAttribute->getOptions() as $option ) {
    if ( $option->getValue() ) {
        $attributeValues[] = [
            'label' => $option->getLabel(),
            'attribute_id' => $configAttribute->getId(),
            'value_index' => $option->getValue() ];
    }
}

$configurableOptionsData = [];
$configurableOptionsData[] = [
    'attribute_id' => $configAttribute->getId(),
    'code' => $configAttribute->getAttributeCode(),
    'label' => $configAttribute->getStoreLabel(),
    'values' => $attributeValues ];

$options = $optionsFactory->create( $configurableOptionsData );
$configProduct->getExtensionAttributes()->setConfigurableProductOptions( $options );
$configProduct->getExtensionAttributes()->setConfigurableProductLinks( $childrenIds );
$resourceModel->save( $configProduct );
```
