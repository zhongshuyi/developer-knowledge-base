---
order: 99

author: 钟舒艺
---
# 整合 mapstruct

## 信息来源

[GitHub](https://github.com/mapstruct/mapstruct)
[官网](https://mapstruct.org/)
[官方文档](https://mapstruct.org/documentation/stable/reference/html)
掘金 - [MapStruct 使用指南](https://juejin.cn/post/6956190395319451679)

## 使用

### maven 安装

```xml
<properties>
  <org.mapstruct.version>1.5.3.Final</org.mapstruct.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>${org.mapstruct.version}</version>
  </dependency>
</dependencies>
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.8.1</version>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <annotationProcessorPaths>
          <path>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct-processor</artifactId>
            <version>${org.mapstruct.version}</version>
          </path>
        </annotationProcessorPaths>
      </configuration>
    </plugin>
  </plugins>
</build>
```

如果跟 `Lombok`  一起使用的话需要加入 `lombok-mapstruct-binding` 插件，而且要按一定顺序写

```xml
<properties>
  <org.mapstruct.version>1.5.3.Final</org.mapstruct.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>${org.mapstruct.version}</version>
  </dependency>
</dependencies>
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.8.1</version>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <annotationProcessorPaths>
          <path>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
              <version>1.18.24</version>
          </path>
          <!-- mapstruct注解处理器 -->
          <path>
              <groupId>org.mapstruct</groupId>
              <artifactId>mapstruct-processor</artifactId>
              <version>1.5.3.Final</version>
          </path>
          <!-- lombok 与 mapstruct 组合插件-->
          <path>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok-mapstruct-binding</artifactId>
              <version>0.2.0</version>
          </path>
        </annotationProcessorPaths>
      </configuration>
    </plugin>
  </plugins>
</build>
```

> `Lombok` 必须在 `mapstruct` 之前，而且在 `annotationProcessorPaths` 必须配置 `Lombok` （一般使用直接在 `dependencies` 中配置即可），在 `Lombok 1.18.16` 及更高版本时需要加入插件 `lombok-mapstruct-binding` 并在他们之后。因为 `maven-compiler-plugin` 应该是按顺序来执行 `annotationProcessorPaths` 下的节点

### IDEA 插件

IDEA 内安装插件 `MapStruct support` ，如果不安装的话多个源参数的映射时 IDEA 会爆红

### 简单使用

如果所有字段名称都相同
为了在这两者之间进行映射，我们要创建一个 `DoctorMapper` 接口。对该接口使用 `@Mapper` 注解，`MapStruct` 就会知道这是两个类之间的映射器。

```java
import org.mapstruct.Mapper;
import org.mapstruct.factory.Mappers;
...

@Mapper
    public interface DoctorMapper {
        DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);
        DoctorDto toDto(Doctor doctor);
    }

```

当我们构建/编译应用程序时，MapStruct注解处理器插件会识别出DoctorMapper接口并为其生成一个实现类。

```java
public class DoctorMapperImpl implements DoctorMapper {
    @Override
    public DoctorDto toDto(Doctor doctor) {
        if ( doctor == null ) {
            return null;
        }
        DoctorDtoBuilder doctorDto = DoctorDto.builder();

        doctorDto.id(doctor.getId());
        doctorDto.name(doctor.getName());

        return doctorDto.build();
    }
}
```

使用的时候就可以这样使用

```java
DoctorDto doctorDto = DoctorMapper.INSTANCE.toDto(doctor);
```

### 不同字段

```java
@Mapper
public interface DoctorMapper {
    DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

    @Mapping(source = "specialty", target = "specialization")
    DoctorDto toDto(Doctor doctor);
}
```

这个注解代码的含义是：`Doctor` 中的 `specialty`字段对应于 `DoctorDto` 类的 `specialization`
`source` ：来源
`target` ：目标

### 多个源类

IDEA 内安装插件 `MapStruct support` ，如果不安装的话多个源参数的映射时 IDEA 会爆红

```java
@Mapper
public interface AddressMapper {

    @Mapping(target = "description", source = "person.description")
    @Mapping(target = "houseNumber", source = "address.houseNo")
    DeliveryAddressDto personAndAddressToDeliveryAddressDto(Person person, Address address);
}
```

如果多个源对象内定义了相同名称的属性，则必须使用 `@Mapping`映射，不然会报错

## 封装

如果每次转换都要新建个借口加上写大量的方法定义，那也太麻烦了吧，所以我们可以通过泛型封装一下

新建一个公共接口

```java
import java.util.List;

/**
 * .mapstruct 转换的公共接口。
 *
 * @param <V> 视图层 VO 类
 * @param <D> 数据传输层 DTO 类
 * @param <E> 实体层 Entity 类
 * @author 钟舒艺
 * @since 2022-10-22 13:56
 */
public interface BaseConvert<V, D, E> {

    /**
     * 转化为 VO.
     *
     * @param entity 实体
     * @return VO
     */
    V convertToVO(E entity);

    /**
     * 转换为 VO 集合。
     *
     * @param list 实体集合
     * @return VO 集合
     */
    List<V> convertToVOList(List<E> list);

    /**
     * 转化为 DTO.
     *
     * @param entity 实体
     * @return DTO
     */
    D convertToDTO(E entity);

    /**
     * 转换为 DTO 集合。
     *
     * @param list 实体集合
     * @return DTO 集合
     */
    List<D> convertToDTOList(List<E> list);

    /**
     * VO 转换为实体。
     *
     * @param vo vo
     * @return 实体
     */
    E voToEntity(V vo);


    /**
     * DTO 转换为实体。
     *
     * @param dto 数据传输对象
     * @return 实体对象
     */
    E dtoToEntity(D dto);

    /**
     * VO 集合转化为 实体对象集合。
     *
     * @param voList 视图对象集合
     * @return 实体对象集合。
     */
    List<E> voListToEntityList(List<V> voList);

    /**
     * DTO 集合转化为 实体对象集合。
     *
     * @param dto 视图对象集合
     * @return 实体对象集合。
     */
    List<E> dtoListToEntityList(List<V> dto);
}
```

使用的话直接继承这个公共接口

```java
package com.ling.system.convert;

import com.ling.common.core.interfaces.BaseConvert;
import com.ling.system.dto.SysAdminDTO;
import com.ling.system.entity.SysAdmin;
import com.ling.system.vo.SysAdminVO;
import org.mapstruct.Mapper;
import org.mapstruct.factory.Mappers;

/**
 * 系统用户转换类。
 *
 * @author 钟舒艺
 * @since 2022-10-22 11:09
 **/
@Mapper
public interface SysAdminConvert extends BaseConvert<SysAdminVO, SysAdminDTO, SysAdmin> {

    /**
     * 实例。
     */
    SysAdminConvert INSTANCT = Mappers.getMapper(SysAdminConvert.class);
}
```

代码中使用

```java
SysAdminConvert.INSTANCT.convertToDTO(sysAdmin);
```
