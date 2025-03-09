# C# 中的规范模式

- 如果您曾经对复杂的过滤逻辑感到困惑，或者淹没在 if-else 语句中，那么让我向您介绍一种设计模式：**规范模式 (Specification Pattern)**

- 规范模式是一种行为设计模式，允许您将业务规则和标准封装到可重用的物件（称为规范）中。每个规范代表一个特定的规则或条件，可以是：
  - 简单（单一条件）
  - 复合（使用逻辑运算与其他规范组合）

- 这种模式创建可以组合、重用和扩展的规范，在于能够在运行时动态组合规范，使您能够灵活地适应不同的过滤或验证需求，而不会使您的程式码库充斥着条件逻辑
- 优点：
  - **可重用性**：一次编写，随处使用。规范是模组化的，可以在整个应用程式中重用
  - **灵活性**：动态组合多个规范，而无需更改核心类别
  - **可读性**：我们获得清晰、不言自明的规范，而不是多个复杂的条件
  - **可测试性**：规范可以独立进行单元测试，使业务规则易于验证和维护

## 实现范例

### 步骤 1：定义实体类别

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public bool InStock { get; set; }
    public string Category { get; set; }
}
```

### 步骤 2：创建规范介面

```csharp
public interface ISpecification<T>
{
    bool IsSatisfiedBy(T entity);
}
```

### 步骤 3：构建具体规范

```csharp
public class PriceSpecification : ISpecification<Product>
{
    private readonly decimal _minPrice;
    private readonly decimal _maxPrice;

    public PriceSpecification(decimal minPrice, decimal maxPrice)
    {
        _minPrice = minPrice;
        _maxPrice = maxPrice;
    }

    public bool IsSatisfiedBy(Product product)
    {
        return product.Price >= _minPrice && product.Price <= _maxPrice;
    }
}

public class InStockSpecification : ISpecification<Product>
{
    public bool IsSatisfiedBy(Product product)
    {
        return product.InStock;
    }
}

public class CategorySpecification : ISpecification<Product>
{
    private readonly string _category;

    public CategorySpecification(string category)
    {
        _category = category;
    }

    public bool IsSatisfiedBy(Product product)
    {
        return product.Category.Equals(_category, StringComparison.OrdinalIgnoreCase);
    }
}
```

### 步骤 4：创建复合规范

```csharp
public class AndSpecification<T> : ISpecification<T>
{
    private readonly ISpecification<T> _spec1;
    private readonly ISpecification<T> _spec2;

    public AndSpecification(ISpecification<T> spec1, ISpecification<T> spec2)
    {
        _spec1 = spec1;
        _spec2 = spec2;
    }

    public bool IsSatisfiedBy(T entity)
    {
        return _spec1.IsSatisfiedBy(entity) && _spec2.IsSatisfiedBy(entity);
    }
}

public class OrSpecification<T> : ISpecification<T>
{
    private readonly ISpecification<T> _spec1;
    private readonly ISpecification<T> _spec2;

    public OrSpecification(ISpecification<T> spec1, ISpecification<T> spec2)
    {
        _spec1 = spec1;
        _spec2 = spec2;
    }

    public bool IsSatisfiedBy(T entity)
    {
        return _spec1.IsSatisfiedBy(entity) || _spec2.IsSatisfiedBy(entity);
    }
}

public class NotSpecification<T> : ISpecification<T>
{
    private readonly ISpecification<T> _spec;

    public NotSpecification(ISpecification<T> spec)
    {
        _spec = spec;
    }

    public bool IsSatisfiedBy(T entity)
    {
        return !_spec.IsSatisfiedBy(entity);
    }
}
```

### 步骤 5：应用规范

```csharp
public static void Main()
{
    var products = new List<Product>
    {
        new Product { Id = 1, Name = "Laptop", Price = 1200, InStock = true, Category = "Electronics" },
        new Product { Id = 2, Name = "Phone", Price = 800, InStock = false, Category = "Electronics" },
        new Product { Id = 3, Name = "Shoes", Price = 150, InStock = true, Category = "Apparel" },
        new Product { Id = 4, Name = "TV", Price = 700, InStock = true, Category = "Electronics" }
    };

    var inStockSpec = new InStockSpecification();
    var electronicsCategorySpec = new CategorySpecification("Electronics");
    var priceRangeSpec = new PriceSpecification(500, 1000);

    var inStockAndElectronicsSpec = new AndSpecification<Product>(inStockSpec, electronicsCategorySpec);
    var finalSpec = new AndSpecification<Product>(inStockAndElectronicsSpec, priceRangeSpec);

    var filteredProducts = products.Where(p => finalSpec.IsSatisfiedBy(p)).ToList();

    foreach (var product in filteredProducts)
    {
        Console.WriteLine($"Product: {product.Name}, Price: {product.Price}, In Stock: {product.InStock}");
    }
}
```

预期输出：

```
Product: TV, Price: 700, In Stock: True
```

最终规范 (finalSpec) 仅包含以下产品：

*   有库存
*   属于电子类别
*   价格在 500 美元到 1000 美元之间
