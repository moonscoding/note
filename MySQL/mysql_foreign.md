## 외래키

MySQL에서 외래키는 InnoDB 스토리지 엔진에서만 생성

외래키 제약이 설정되면 자동으로 연관되는 테이블 칼럼에 인덱스까지 생성

외래키가 제거되지 않은 상태에서 자동으로 생성된 인덱스를 삭제할 수 없음



>  InnoDB 외래키 관리의 특징

- 데이블의 변경(쓰기잠금)이 발생하는 경우에만 잠금 경합( 잠금 대기)이 발생
- 외래키와 연관되지 않은 칼럼의 변경은 최대한 잠금 경합( 잠금 대기)를 발생시키지 않습니다.

```sql
CREATE TABLE tb_parent (
    id INT NOT NULL,
    fd VARCHAR(100) NOT NULL,
    PRIMARY KEY (id)
) ENGINE=INNODB;

CREATE TABLE tb_child (
    id INT NOT NULL,
    pid INT DEFAULT NULL, -- // parent.id 칼럼 참조
    fd VARCHAR(100) DEFAULT NULL,
    PRIMARY KEY (id),
    KEY ix_parentid (pid),
    CONSTRAINT child_ibfk_1 FOREIGN KEY (pid) REFERENCES tb_parent (id) ON DELETE CASCADE
) ENGINE-INNODB;

INSERT INTO tb_parent VALUES (1, 'parent-1'), (2, 'parent-2');
INSERT INTO tb_parent VALUES (100, 1, 'child-100');
```

