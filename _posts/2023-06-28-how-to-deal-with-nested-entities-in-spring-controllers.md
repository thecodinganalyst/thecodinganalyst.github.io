---
title: "How to deal with nested entities in Spring controllers"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Spring
  - Java
---

Creating a webservice with Spring is almost trivial, you usually won't go wrong by following the [tutorials](https://www.thecodinganalyst.com/tutorial/Spring-boot-application-getting-started/). However, things can get tricky sometimes when you want more features out of it. One example is when you are having nested entities like a father and sons relationship.

![father and sons relationship](/assets/images/2023/06/father-son.png)

Using Spring Boot and Spring Data, the entities can be declared as such.

```
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Father {
    @Id
    @GeneratedValue
    Long id;
    String name;
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy = "father", orphanRemoval = true)
    @Builder.Default
    List<Son> sonList = new ArrayList<>();

}
```

```
@Data
@Entity
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Son {
    @Id
    @GeneratedValue
    Long id;
    String name;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    @JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id", scope = Father.class)
    Father father;
}
```

But when we try to do a test, it might even seem ok when getting the result from the insert, but not when we try to use the get to get the inserted result again from the controller.

```
@Test
void create_willFail() throws Exception {
    Son son1 = Son.builder().name("son 1").build();
    Son son2 = Son.builder().name("son 2").build();
    Father father = Father.builder().name("father").sonList(List.of(son1, son2)).build();

    MvcResult insertResult = mvc.perform(post("/fathers")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(father)))
            .andExpect(status().isCreated())
            .andReturn();
    String insertResultJson = insertResult.getResponse().getContentAsString();
    Father insertedFather = objectMapper.readValue(insertResultJson, Father.class);
    assertThat(insertedFather.getSonList().size(), is(2));

    MvcResult getResult = mvc.perform(get("/fathers/" + insertedFather.getId())
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(father)))
            .andExpect(status().isOk())
            .andReturn();
    String getResultJson = getResult.getResponse().getContentAsString();
    Father gotFather = objectMapper.readValue(getResultJson, Father.class);
    assertThat(gotFather.getSonList().size(), is(0));
}
```

The reason we get the 2 sons from the first assertion is because we already added the 2 sons in the father object which we have sent to the controller. But because the sons doesn't have the `father` field set, the `father` field of the 2 sons are empty. And since the foreign key is at the son, the relationship isn't persisted. 

One might think an easy solution is to set the `father` field in the son explicitly, but it will still fail with a stackoverflow. It happens because when the objectMapper will be recursively serializing the sonList and father relationships. Plus, it doesn't feel intuitive for the api user to set the father field in the sons, when the sonList of the father field is already populated.

```
@Test
void create_willOverflow(){
    Son son1 = Son.builder().name("son 1").build();
    Son son2 = Son.builder().name("son 2").build();
    Father father = Father.builder().name("father").sonList(List.of(son1, son2)).build();
    son1.setFather(father);
    son2.setFather(father);

    assertThrows(JsonMappingException.class, () -> objectMapper.writeValueAsString(father));
}
```

Let's do it again with a `ParentClass` and `ChildClass`. 

```
@Data
@Entity
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ParentClass {
    @Id
    @GeneratedValue
    Long id;
    String name;
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy = "parent", orphanRemoval = true)
    @Builder.Default
    List<ChildClass> children = new ArrayList<>();
}
```

```
@Data
@Entity
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ChildClass {
    @Id
    @GeneratedValue
    Long id;
    String name;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    @ToString.Exclude
    ParentClass parent;
}
```

Instead of using our domain class directly in the controllers, it is better to create DTOs ([Data Transfer Objects](https://martinfowler.com/eaaCatalog/dataTransferObject.html)) for the entities class instead, so that we can also hide certain information of our entities that shouldn't be known by the api users. 

```
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ParentClassDto {
    @EqualsAndHashCode.Exclude
    Long id;
    String name;
    List<ChildClassDto> children;

    public ParentClass toParentClass(){
        ParentClass parentClass = ParentClass.builder()
                .id(id)
                .name(name)
                .children(ChildClassDto.fromChildClassDtoList(children))
                .build();
        parentClass.getChildren().forEach(child -> child.setParent(parentClass));
        return parentClass;
    }

    public static ParentClassDto fromParentClass(ParentClass parentClass){
        return ParentClassDto.builder()
                .id(parentClass.getId())
                .name(parentClass.getName())
                .children(parentClass.getChildren().stream().map(ChildClassDto::fromChildClass).toList())
                .build();
    }
}
```

```
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ChildClassDto {
    @EqualsAndHashCode.Exclude
    Long id;
    String name;

    public ChildClass toChildClass(){
        return ChildClass.builder()
                .id(id)
                .name(name)
                .build();
    }

    public static ChildClassDto fromChildClass(ChildClass childClass){
        return ChildClassDto.builder()
                .id(childClass.getId())
                .name(childClass.getName())
                .build();
    }

    public static List<ChildClass> fromChildClassDtoList(List<ChildClassDto> childClassDtoList){
        return childClassDtoList.stream()
                .map(ChildClassDto::toChildClass)
                .toList();
    }
}
```

In the DTOs, we can also provide the mapping methods to map the entities to DTOs, and vice versa. We can also use dedicated mapper class instead, and use Java Records for the DTOs, which makes it even cleaner. But we shall stick to this for now, so that we have less files to manage.

I also added the [`@EqualsAndHashCode.Exclude`](https://projectlombok.org/features/EqualsAndHashCode) to the id fields, so that I can simply compare 2 DTOs just on the other value fields for equality, in order not to compare the fields one by one in my tests. 

Then in my controller, I will only accept and return the DTOs. So I'll have to convert the DTOs to my entity objects when receiving the values from the api calls, and convert them from entity objects back to DTOs when I return the results. 

```
@RestController
public class ParentClassController {
    ParentClassRepository parentClassRepository;

    public ParentClassController(ParentClassRepository parentClassRepository){
        this.parentClassRepository = parentClassRepository;
    }

    @GetMapping
    public List<ParentClassDto> list(){
        return parentClassRepository.findAll()
                .stream().
                map(ParentClassDto::fromParentClass)
                .toList();
    }

    @GetMapping("/{id}")
    public ParentClassDto get(@PathVariable("id") String id){
        ParentClass parentClass = parentClassRepository.findById(Long.valueOf(id))
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
        return ParentClassDto.fromParentClass(parentClass);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ParentClassDto create(@RequestBody ParentClassDto parentClassDto){
        ParentClass parentClass = parentClassRepository.save(parentClassDto.toParentClass());
        return ParentClassDto.fromParentClass(parentClass);
    }

    @PutMapping("/{id}")
    public ParentClassDto update(@PathVariable("id") String id, @RequestBody ParentClassDto parentClassDto){
        ParentClass parentClass = parentClassDto.toParentClass();

        ParentClass retrievedParentClass = parentClassRepository.findById(Long.valueOf(id))
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));

        retrievedParentClass.setName(parentClassDto.getName());

        retrievedParentClass.getChildren().clear();
        List<ChildClass> updatedChildren = new ArrayList<>(parentClass.getChildren());
        updatedChildren.forEach(childClass -> childClass.setParent(retrievedParentClass));
        retrievedParentClass.getChildren().addAll(updatedChildren);

        ParentClass updatedParentClass = parentClassRepository.save(retrievedParentClass);
        return ParentClassDto.fromParentClass(updatedParentClass);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable("id") String id){
        parentClassRepository.deleteById(Long.valueOf(id));
    }
}
```

So now, I just have to add the children DTOs in my parent DTO, and deal only with the parent DTO when using the api. 

```
@SpringBootTest
@AutoConfigureMockMvc
class ParentClassControllerTest {
    @Autowired
    MockMvc mvc;
    @Autowired
    ObjectMapper objectMapper;

    @Test
    void update() throws Exception {
        // First create the nested entities
        ChildClassDto child1 = ChildClassDto.builder()
                .name("child1")
                .build();
        ChildClassDto child2 = ChildClassDto.builder()
                .name("child2")
                .build();
        ParentClassDto parent = ParentClassDto.builder()
                .name("Parent")
                .children(List.of(child1, child2))
                .build();

        MvcResult insertResult = mvc.perform(post("/")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(parent)))
                .andExpect(status().isCreated())
                .andReturn();
        String insertResultJson = insertResult.getResponse().getContentAsString();
        ParentClassDto insertedParent = objectMapper.readValue(insertResultJson, ParentClassDto.class);

        // Then update
        ChildClassDto child1Update = ChildClassDto.builder().name("Updated Child 1").build();
        ChildClassDto child2Update = ChildClassDto.builder().name("Updated Child 2").build();
        ParentClassDto parentUpdate = ParentClassDto.builder()
                .name("Updated Parent")
                .children(List.of(child1Update, child2Update))
                .build();

        MvcResult updateResult = mvc.perform(put("/" + insertedParent.getId().toString())
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(parentUpdate)))
                .andExpect(status().isOk())
                .andReturn();
        String updateResultJson = updateResult.getResponse().getContentAsString();
        ParentClassDto updatedParent = objectMapper.readValue(updateResultJson, ParentClassDto.class);

        assertThat(updatedParent, is(parentUpdate));

        // Make sure next get still gets the same result
        MvcResult getResult = mvc.perform(get("/" + insertedParent.getId().toString())
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(parentUpdate)))
                .andExpect(status().isOk())
                .andReturn();
        String getResultJson = getResult.getResponse().getContentAsString();
        ParentClassDto gotParent = objectMapper.readValue(getResultJson, ParentClassDto.class);

        assertThat(gotParent, is(updatedParent));
    }
}
```

A working copy of the above example is available on [https://github.com/thecodinganalyst/SpringDataNestedEntityDemo](https://github.com/thecodinganalyst/SpringDataNestedEntityDemo).