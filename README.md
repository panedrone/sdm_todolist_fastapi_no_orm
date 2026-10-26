A project demonstrating how to use [SQL DAL Maker](https://github.com/panedrone/sqldalmaker) + Python, FastAPI, and
no-ORM-scenario.

Front-end is written in Vue.js 2.7. RDBMS is SQLite3.

The code in `dbal` can also be generated without an IDE by [sdmc](https://github.com/panedrone/sdmc),
the command-line twin of the plug-ins: in the project folder, `sdmc gen` regenerates it and `sdmc val`
checks that it is up to date.

![sdm-todo-app.png](sdm-todo-app.png)

sdm.xml

```xml

<sdm>

    <dto-class name="Project" ref="projects"/>

    <dto-class name="ProjectLi" ref="get_projects.sql"/>

    <dto-class name="Task" ref="tasks"/>

    <dto-class name="TaskLi" ref="get_project_tasks.sql"/>

    <dao-class name="ProjectsDao">
        <crud dto="Project"/>
        <query-dto-list dto="ProjectLi" method="get_projects"/>
    </dao-class>

    <dao-class name="TasksDao">
        <crud dto="Task"/>
        <query-dto-list dto="TaskLi" method="get_project_tasks(p_id)"/>
    </dao-class>

</sdm>
```

Generated code in action:

```python
from typing import List
from fastapi import Depends, FastAPI
from dbal.db import get_ds
from dbal.data_store import DataStore
from dbal.project import Project
from dbal.projects_dao import ProjectsDao
from schemas import *

app = FastAPI(title="SDM + FastAPI, no-ORM, SQLite3",
              description="Quick Demo of how to use SQL DAL Maker + FastAPI, no-ORM, SQLite3",
              version="1.0.0", )


@app.get('/api/projects', tags=["ProjectList"], response_model=List[SchemaProjectLi])
def get_all_projects(ds: DataStore = Depends(get_ds)):
    return ProjectsDao(ds).get_projects()


@app.post('/api/projects', tags=["ProjectList"], status_code=201)
async def project_create(item_request: SchemaProjectCreateUpdate, ds: DataStore = Depends(get_ds)):
    project = Project()
    project.p_name = item_request.p_name
    ProjectsDao(ds).create_project(project)
    ds.commit()


@app.get('/api/projects/{p_id}', tags=["Project"], response_model=SchemaProject)
def project_read(p_id: int, ds: DataStore = Depends(get_ds)):
    pr = Project()
    ProjectsDao(ds).read_project(p_id, pr)
    return pr


@app.put('/api/projects/{p_id}', tags=["Project"])
async def project_update(p_id: int, item_request: SchemaProjectCreateUpdate, ds: DataStore = Depends(get_ds)):
    pr = Project()
    pr.p_id = p_id
    pr.p_name = item_request.p_name
    ProjectsDao(ds).update_project(pr)
    ds.commit()


@app.delete('/api/projects/{p_id}', tags=["Project"], status_code=204)
async def project_delete(p_id: int, ds: DataStore = Depends(get_ds)):
    ProjectsDao(ds).delete_project(p_id)
    ds.commit()
```
