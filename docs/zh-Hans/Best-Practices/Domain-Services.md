## 领域服务最佳实践 & 约定

### 介绍

在领域驱动设计（DDD）解决方案中，核心业务逻辑通常在聚合（实体）和领域服务中实现。在以下情况中尤其需要创建领域服务：

- 您实现了核心领域逻辑，其取决于某些服务（如存储库或其他外部服务）。

- 您需要实现的逻辑与多于一个聚合/实体相关，因此它无法适合任何聚合。

### ABP Domain Service Infrastructure
领域服务很简单，无状态类。虽然您不必继承任何服务或界面，但ABP框架提供了一些有用的基类和约定。

#### DomainService & IDomainService
要么从`DomainService`基类推出领域服务，要么直接实现`IDomainService`接口。

**例子:从 `DomainService`继承创建领域服务类.**

````C#
using Volo.Abp.Domain.Services;

namespace MyProject.Issues
{
    public class IssueManager : DomainService
    {
        
    }
}
````
当你这样做;

ABP框架自动将类注册到具有瞬态实例的依赖注入系统。

您可以直接使用某些常见服务作为基础属性，而无需手动注入（例如 ILogger和IGuidGenerator）。

**例子：实现为用户分配问题的领域逻辑

````C#
public class IssueManager : DomainService
{
    private readonly IRepository<Issue, Guid> _issueRepository;

    public IssueManager(IRepository<Issue, Guid> issueRepository)
    {
        _issueRepository = issueRepository;
    }
    
    public async Task AssignAsync(Issue issue, AppUser user)
    {
        var currentIssueCount = await _issueRepository
            .CountAsync(i => i.AssignedUserId == user.Id);
        
        //Implementing a core business validation
        if (currentIssueCount >= 3)
        {
            throw new IssueAssignmentException(user.UserName);
        }

        issue.AssignedUserId = user.Id;
    }    
}


````

