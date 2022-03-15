# HQL.-ManyToOne-Joint

                                                         Using TypedQuery

@Entity
public class Employee {
  @Id
  @GeneratedValue
  private long id;
  private String name;
  private long salary;
  @ManyToOne(cascade = CascadeType.ALL)
  private Department dept;
    .............
  public static Employee create(String name, int salary, Department dept) {
      Employee e = new Employee();
      e.setName(name);
      e.setSalary(salary);
      e.setDept(dept);
      dept.addEmployee(e);
      return e;
  }
    .............
}

xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx


@Entity
public class Department {
  @Id
  @GeneratedValue
  private long id;
  private String name;
  @OneToMany(mappedBy = "dept")
  private List<Employee> employees;
    .............
  public void addEmployee(Employee e) {
      if (this.employees == null) {
          employees = new ArrayList<>();
      }
      employees.add(e);
  }
    .............
}

xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

public class ExampleMain {
  private static EntityManagerFactory entityManagerFactory =
          Persistence.createEntityManagerFactory("example-unit");

  public static void main(String[] args) {
      try {
          persistEmployees();
          findEmployees();
          findEmployeeSalaries();
          findDepartments();
          findHighestSalaryDept();
      } finally {
          entityManagerFactory.close();
      }
  }

  public static void persistEmployees() {
      Department deptIt = new Department();
      deptIt.setName("IT");

      Department deptAdmin = new Department();
      deptAdmin.setName("Admin");

      Employee employee1 = Employee.create("Diana", 3000, deptIt);
      Employee employee2 = Employee.create("Rose", 4000, deptAdmin);
      Employee employee3 = Employee.create("Denise", 1500, deptIt);
      Employee employee4 = Employee.create("Mike", 2000, deptAdmin);
      EntityManager em = entityManagerFactory.createEntityManager();
      em.getTransaction().begin();
      em.persist(employee1);
      em.persist(employee2);
      em.persist(employee3);
      em.persist(employee4);
      em.getTransaction().commit();
      em.close();
  }

  private static void findEmployees() {
      System.out.println("-- all Employees--");
      EntityManager em = entityManagerFactory.createEntityManager();
      TypedQuery<Employee> query = em.createQuery("SELECT e FROM Employee e", Employee.class);
      List<Employee> resultList = query.getResultList();
      resultList.forEach(System.out::println);
  }

  private static void findEmployeeSalaries() {
      System.out.println("-- Employee salaries --");
      EntityManager em = entityManagerFactory.createEntityManager();
      TypedQuery<Long> query = em.createQuery("SELECT e.salary FROM Employee e", Long.class);
      List<Long> resultList = query.getResultList();
      resultList.forEach(System.out::println);
  }

  private static void findDepartments() {
      System.out.println("-- All dept departments --");
      EntityManager em = entityManagerFactory.createEntityManager();
      TypedQuery<Department> query = em.createQuery("SELECT d FROM Department d", Department.class);
      List<Department> resultList = query.getResultList();
      resultList.forEach(System.out::println);
  }

  private static void findHighestSalaryDept() {
      System.out.println("-- Dept with max salary --");
      EntityManager em = entityManagerFactory.createEntityManager();
      TypedQuery<String> query = em.createQuery("SELECT d.name FROM Department d "
                      + " JOIN d.employees e where e.salary = (SELECT MAX(e2.salary) FROM Employee e2) ",
              String.class);
      String dept = query.getSingleResult();
      System.out.println(dept);
  }
}
