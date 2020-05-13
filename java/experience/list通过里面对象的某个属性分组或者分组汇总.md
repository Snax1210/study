1，跟据某个属性分组OfficeId

    Map<String, List<IncomeSumPojo>> collect = list.stream().collect(Collectors.groupingBy(IncomeSumPojo::getOfficeId));

2，根据某个属性分组OfficeId，汇总某个属性Money

    Map<String, Double> collect = list.stream().collect(Collectors.groupingBy(IncomeSumPojo::getOfficeId,Collectors.summingDouble(IncomeSumPojo::getMoney)));

3，根据某个属性添加条件过滤数据，

    list = list.stream().filter(u -> !u.getAmount().equals("0.00")).collect(Collectors.toList());

4，判断一组对象里面有没有属性值是某个值

    List<Menu> menuList = UserUtils.getMenuList();
    boolean add = menuList.stream().anyMatch(m -> "plan:ctPlan:add".equals(m.getPermission()));

5，取出一组对象的某个属性组成一个新集合

    List<String> tableNames=list.stream().map(User::getMessage).collect(Collectors.toList());

6，list去重复

    list = list.stream().distinct().collect(Collectors.toList());
