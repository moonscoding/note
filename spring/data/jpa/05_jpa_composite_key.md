# 복합키(Composite Key) 매핑

## Spring JPA

### Embedded

#### Key Class

- getter / setter

```java
package com.mo.guard.model.embedded;

import javax.persistence.Column;
import javax.persistence.Embeddable;
import java.io.Serializable;

@Embeddable
@Data
public class RelationAuthResourceId implements Serializable {

    @Column(name = "auth_sq", insertable = false, updatable = false)
    int authSequence;

    @Column(name = "resource_sq", insertable = false, updatable = false)
    int resourceSequence;

    @Override
    public String toString() {
        return authSequence + "-" + resourceSequence;
    }
}
```

#### Entity Class

```java
@Entity
@Table(name = "G_R_AUTH_RESOURCE_TB")
@EntityListeners(value = {AuditingEntityListener.class})
@Data
public class RelationAuthResource {

    @EmbeddedId
    public RelationAuthResourceId pk;

    @Column(name = "auth_sq", insertable = false, updatable = false)
    public int authSequence;

    @Column(name = "resource_sq", insertable = false, updatable = false)
    public int resourceSequence;
    
}
```



### IdClass

#### Key Class

```

```

#### Entity Class

```

```

