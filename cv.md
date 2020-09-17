
# **Rebkavets Aliaksei**
*+375 (29) 870-42-78  — preferred means of communication*
*lehichh@gmail.com*
### Responsible for the support and development of logistics software, my functions are:
* analysis and search for solutions to the tasks set by clients;
* development of new and improvement of existing functionality;
* development of software product code according to ready-made specifications;
* formation of internal documentation based on the results of the work;
* code analysis and optimization to improve the quality and performance of software.
### Knowledge:
* Java Core;
* JUnit;
* JDBC;
* Servlets;
* JSP;
* Spring IoC;
* Spring MVC;
* PostgreSQL.
### Code
    package main.java.service;
    import main.java.model.DataSet;
    import main.java.model.DatabaseConnectionRepository;
    import main.java.model.DatabaseManager;
    import main.java.model.UserActionRepository;
    import main.java.model.entity.DatabaseConnection;
    import main.java.model.entity.UserAction;
    import main.java.model.UserActionsRepository;
    import main.java.model.entity.UserActions;
    import main.java.model.impl.DataSetImpl;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;
    import java.util.ArrayList;
    import java.util.LinkedList;
    import java.util.List;
    @Component
    public abstract class ServiceImpl implements Service {
    private List<String> commandsList;
    public abstract DatabaseManager getManager();
    protected abstract DatabaseManager getManager();
    @Autowired
    private UserActionsRepository userActionsDao;
    @Autowired
    private DatabaseConnectionRepository databaseConnections;
    private UserActionRepository userActions;
    @Override
    public List<String> getCommandsList() {
        return commandsList;
    }
    @Override
    public DatabaseManager connect(String databaseName, String userName, String password) {
        DatabaseManager manager = getManager();
        manager.connect(databaseName, userName, password);
        saveAction(databaseName, userName, "CONNECT");
        userActions.createAction(databaseName, userName, "CONNECT");
        return manager;
    }
    private void saveAction(String databaseName, String userName, String action) {
        DatabaseConnection databaseConnection = databaseConnections.findByUserNameAndDbName(userName, databaseName);
        if (databaseConnection == null) {
            databaseConnection = databaseConnections.save(new DatabaseConnection(userName, databaseName));
        }
        userActionsDao.save(new UserAction(action, databaseConnection));
    }
    @Override
    public List<List<String>> find(DatabaseManager manager, String tableName) {
        List<List<String>> result = new LinkedList<>();
        List<DataSet> tableData = manager.getAllTableData(tableName);
        List<String> columns = new LinkedList<>(manager.getTableColumns(tableName));
        result.add(columns);
        for (DataSet dataSet : tableData) {
            List<String> row = new ArrayList<>(columns.size());
            result.add(row);
            for (String column : columns) {
                row.add(dataSet.get(column).toString());
            }
        }
        saveAction(manager.getDatabaseName(), manager.getUserName(), "FIND (" + tableName + ")");
        userActions.createAction(manager.getDatabaseName(), manager.getUserName(), "FIND (" + tableName + ")");
        return result;
    }
    @Override
    public void clear(DatabaseManager manager, String tableName) {
        manager.clear(tableName);
        userActionsDao.save(new UserAction("CLEAN (" + tableName + ")", new DatabaseConnection(manager.getUserName(), manager.getDatabaseName())));
        userActions.save(new UserActions("CLEAN (" + tableName + ")", new DatabaseConnection(manager.getUserName(), manager.getDatabaseName())));
    }
    @Override
    public List<String> getTablesNames(DatabaseManager manager) {
        saveAction(manager.getDatabaseName(), manager.getUserName(), "Get table names");
        userActions.createAction(manager.getDatabaseName(), manager.getUserName(), "Get table names");
        return new LinkedList<>(manager.getTableNames());
    }
    @Override
    public void drop(DatabaseManager manager, String tableName) {
        manager.drop(tableName);
        saveAction(manager.getDatabaseName(), manager.getUserName(), "DROP (" + tableName + ")");
        userActions.createAction(manager.getDatabaseName(), manager.getUserName(), "DROP (" + tableName + ")");
    }
    @Override
    public void create(DatabaseManager manager, String tableName, List<String> listColumn) {
        manager.create(tableName, listColumn);
        saveAction(manager.getDatabaseName(), manager.getUserName(), "CREATE (" + tableName + ")");
        userActions.createAction(manager.getDatabaseName(), manager.getUserName(), "CREATE (" + tableName + ")");
    }
    @Override
    public List<String> getColumnsTable(DatabaseManager manager, String tableName) {
        saveAction(manager.getDatabaseName(), manager.getUserName(), "Get columns name (" + tableName + ")");
        userActions.createAction(manager.getDatabaseName(), manager.getUserName(), "Get columns name (" + tableName + ")");
        return new LinkedList<>(manager.getTableColumns(tableName));
    }
    @Override
    public void insert(DatabaseManager manager, String tableName, List<String> listCollumns, List<String> listValues) {
        DataSetImpl dataSet = new DataSetImpl();
        for (int index = 0; index < listCollumns.size(); index++) {
            String columnName = listCollumns.get(index);
            String value = listValues.get(index);
            dataSet.put(columnName, value);
        }
        manager.insert(tableName, dataSet);
        saveAction(manager.getDatabaseName(), manager.getUserName(), "INSERT (" + tableName + ")");
        userActions.createAction(manager.getDatabaseName(), manager.getUserName(), "INSERT (" + tableName + ")");
    }
    @Override
    public void update(DatabaseManager manager, String tableName, String condColName, String condColValue,  List<String> listCollumns, List<String> listValues) {
        List<String> dataSet = new LinkedList<>();
        for (int index = 0; index < listCollumns.size(); index++) {
            String columnName = listCollumns.get(index);
            String value = listValues.get(index);
            if (columnName != null && value != null && !columnName.isEmpty() && !value.isEmpty()) {
                dataSet.add(columnName);
                dataSet.add(value);
            }
        }
        manager.update(tableName, condColName, condColValue, dataSet);
        saveAction(manager.getDatabaseName(), manager.getUserName(), "UPDATE (" + tableName + ")");
        userActions.createAction(manager.getDatabaseName(), manager.getUserName(), "UPDATE (" + tableName + ")");
    }
    @Override
    public void delete(DatabaseManager manager, String deleteTableName, String columnName, String columnValue) {
        manager.delete(deleteTableName, columnName, columnValue);
        saveAction(manager.getDatabaseName(), manager.getUserName(), "DELETE (" + deleteTableName + ") WITH COLUMN ("
        userActions.createAction(manager.getDatabaseName(), manager.getUserName(), "DELETE (" + deleteTableName + ") WITH COLUMN ("
                                            + columnName + ") AND IT VALUE(" + columnValue + ")");
    }
    
    public void setCommandsList(List<String> commandsList) {
        this.commandsList = commandsList;
    }
    @Override
    public  List<UserAction> getAllFor(String userName) throws IllegalAccessException {
    public  List<UserActions> getAllFor(String userName) throws IllegalAccessException {
        if (userName == null) {
            throw new IllegalAccessException("User name can't be null");
        }
        return userActionsDao.findByUserName(userName);
        return userActions.findByUserName(userName);
    }
    }
### Work experience
1. 2014 Школа иностранных языков [Streamline](https://str.by/), English (elementaryA0-intermediateB1);
1. 2020 [Juja](https://edu.juja.com.ua/) Java.

### English language
* IntermediateB1
