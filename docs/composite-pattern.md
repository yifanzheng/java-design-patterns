## 组合模式

组合模式(Composite Design Pattern)，将对象组合成树形结构以表示“部分-整体”的层次结构。组合模式使得用户可以统一单个对象和组合对象的处理逻辑。

组合模式跟面向对象设计中的“组合关系（通过组合来组装两个类）”，完全是两码事。这里讲的“组合模式”，主要是用来处理树形结构数据。这里的“数据”，你可以简单理解为一组对象集合。

正因为其应用场景的特殊性，数据必须能表示成树形结构，这也导致了这种模式在实际的项目开发中并不那么常用。但是，一旦数据满足树形结构，应用这种模式就能发挥很大的作用，能让代码变得非常简洁。

### 组合模式解析

接下来，对于组合模式，我们举个例子来解释一下。

假设有这样一个例子：设计一个类来表示一个学校院系结构，一个学校有多个学院，一个学院有多个专业。在控制台中打印出组织关系。

具体代码如下：

```java
/**
 * OrganizationComponent
 */
public abstract class OrganizationComponent {

	private String name;

	public OrganizationComponent(String name) {
		this.name = name;
	}

	protected abstract void add(OrganizationComponent organizationComponent);
	
	protected abstract void remove(OrganizationComponent organizationComponent);

	protected abstract void print();

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
	
}

/**
 * University
 */
public class University extends OrganizationComponent {

	List<OrganizationComponent> organizationComponents = new LinkedList<>();

	public University(String name) {
		super(name);
	}

	@Override
	protected void add(OrganizationComponent organizationComponent) {
		organizationComponents.add(organizationComponent);
	}

	@Override
	protected void remove(OrganizationComponent organizationComponent) {
		organizationComponents.remove(organizationComponent);
	}

	@Override
	protected void print() {
		System.out.println("--------------" + getName() + "--------------");
		// 遍历 organizationComponents
		for (OrganizationComponent organizationComponent : organizationComponents) {
			organizationComponent.print();
		}
	}

}

/**
 * College 学院
 */
public class College extends OrganizationComponent {

	/**
	 *  存放节点
	 */
	List<OrganizationComponent> organizationComponents = new LinkedList<OrganizationComponent>();

	public College(String name) {
		super(name);
	}

	@Override
	protected void add(OrganizationComponent organizationComponent) {
		//  将来实际业务中，Colleage 的 add 和  University add 不一定完全一样
		organizationComponents.add(organizationComponent);
	}

	@Override
	protected void remove(OrganizationComponent organizationComponent) {
		organizationComponents.remove(organizationComponent);
	}


	@Override
	protected void print() {
		System.out.println("--------------" + getName() + "--------------");
		// 遍历 organizationComponents
		for (OrganizationComponent organizationComponent : organizationComponents) {
			organizationComponent.print();
		}
	}
}

/**
 * Department
 */
public class Department extends OrganizationComponent {

	public Department(String name) {
		super(name);
	}

	@Override
	protected void add(OrganizationComponent organizationComponent) {
		throw new UnsupportedOperationException();
	}

	@Override
	protected void remove(OrganizationComponent organizationComponent) {
       throw new UnsupportedOperationException();
	}

	@Override
	protected void print() {
		System.out.println(this.getName());
	}
}

// 使用举例
public class Main {

	public static void main(String[] args) {
		// 学校
		OrganizationComponent university = new University("清华大学");
		
		// 创建学院
		OrganizationComponent infoEngineerCollege = new College("信息工程学院");

		// 创建各个学院下面的专业
		infoEngineerCollege.add(new Department("软件工程"));
		infoEngineerCollege.add(new Department("网络工程"));
		infoEngineerCollege.add(new Department("计算机科学与技术"));
		
		//将学院加入到学校
		university.add(infoEngineerCollege);

		university.print();
	}
}
```
在上面的代码实现中，对于各组织的打印，也就是 print() 函数，实际上这就是树上的递归遍历算法。对于 Department，我们直接打印其名字。对于 University，我们遍历 University 中每个 College 以及 Department，递归打印出它们的名字。

实际上，组合模式的设计思路，与其说是一种设计模式，不如说是对业务场景的一种数据结构与算法的抽象。其中，数据可以表示成树这种数据结构，业务需求可以通过在树上的递归算法类实现。

### 总结

组合模式，将一组对象组织成树形结构，将单个对象和组合对象都看做树中的节点，以统一处理逻辑，并且它利用树形结构的特点，递归地处理每个子树，依次简化代码实现。使用组合模式的前提在于，你的业务场景必须能够表示成树形结构。所以，组合模式的应用场景也比较局限，它并不是一种很常用的设计模式。



