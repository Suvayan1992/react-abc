/* eslint-disable react-hooks/exhaustive-deps */
import React,{useState,useEffect} from 'react';
import Column from '../components/Column';
import "@atlaskit/css-reset";
import { DragDropContext } from "react-beautiful-dnd";
import initialData from "./initial-data";
import { mutliDragAwareReorder, multiSelectTo as multiSelect } from "../util/util";

const createData = () => {
    let taskIdcount = 0;
    let tasks = Array.from(initialData.tasks);
    let taskIds = Array.from(initialData.tasks);
    for (let i = 0; i < 10; i++) {
        let count = ++taskIdcount;
        let task = {
            id: count,
            content: "Task " + count
        };
        tasks.push(task);
        taskIds.push(task.id);
    }

    return {
        entities: {
        tasks: tasks,
        columns: {
            "column-1": {
                id: "column-1",
                title: "Tasks",
                taskIds: taskIds
            }
        },
        columnOrder: ["column-1"]
      },
      selectedTaskIds: [],
      draggingTaskId: null
    };
}

const App = () => {
    const[state, setState] = useState(createData());

    useEffect(() => {
        window.addEventListener("click", onWindowClick);
        window.addEventListener("keydown", onWindowKeyDown);
        window.addEventListener("touchend", onWindowTouchEnd);
        return() => {
            window.removeEventListener("click", onWindowClick);
            window.removeEventListener("keydown", onWindowKeyDown);
            window.removeEventListener("touchend", onWindowTouchEnd);
        }
    },[])

    useEffect(() => {
        window.addEventListener("click", onWindowClick);
        window.addEventListener("keydown", onWindowKeyDown);
        window.addEventListener("touchend", onWindowTouchEnd);
    },[])

    const onDragStart = (start) => {
        const id = start.draggableId;
        const selected = state.selectedTaskIds.find((taskId) => taskId === id);
        if (!selected) {
            unselectAll();
        }
        setState({
            ...state,
            draggingTaskId: start.draggableId
        });
    };

    const onDragEnd = (result) => {
        const { destination, source, draggableId } = result;
        if (!destination) {
            setState({
                ...state,
                draggingTaskId: null
            });
            return;
        }
        const processed = mutliDragAwareReorder({
            entities: state.entities,
            selectedTaskIds: state.selectedTaskIds,
            source,
            destination
        });
    
        setState({
            ...state,
            ...processed,
            draggingTaskId: null
        });
    
    };

    const onWindowKeyDown = (event) => {
        console.log("onWindowKeyDown", event);
        if (event.defaultPrevented) {
          return;
        }
    
        if (event.key === "Escape") {
          unselectAll();
        }
    };
    
    const onWindowClick = (event) => {
        console.log("onWindowClick", event);
        if (event.defaultPrevented) {
          return;
        }
        unselectAll();
    };
    
    const onWindowTouchEnd = (event) => {
        console.log("onWindowTouchEnd");
        if (event.defaultPrevented) {
          return;
        }
        this.unselectAll();
    };

    const toggleSelection = (taskId) => {
        const selectedTaskIds = state.selectedTaskIds;
        const wasSelected = selectedTaskIds.includes(taskId);
    
        const newTaskIds = (() => {
            if (!wasSelected) {
                return [taskId];
            }
            if (selectedTaskIds.length > 1) {
                return [taskId];
            }
            return [];
        })();
        setState({
            ...state,
            selectedTaskIds: newTaskIds
        });
      };
    
    const toggleSelectionInGroup = (taskId) => {
        const selectedTaskIds = state.selectedTaskIds;
        const index = selectedTaskIds.indexOf(taskId);
    
        // if not selected - add it to the selected items
        if (index === -1) {
            setState({
                ...state,
                selectedTaskIds: [...selectedTaskIds, taskId]
            });
            return;
        }
        const shallow = [...selectedTaskIds];
        shallow.splice(index, 1);
        setState({
            ...state,
            selectedTaskIds: shallow
        });
      };
    

    const multiSelectTo = (newTaskId) => {
        const updated = multiSelect(
            state.entities,
            state.selectedTaskIds,
            newTaskId
        );
        if (updated == null) {
            return;
        }
        setState({
            ...state,
            selectedTaskIds: updated
        });
    };
    
    const unselect = () => {
        unselectAll();
    };
    
    const unselectAll = () => {
        setState({
            ...state,
            selectedTaskIds: []
        });
    };

    return (
        <DragDropContext
            onDragStart={onDragStart}
            onDragEnd={onDragEnd}
        >
        {state.entities.columnOrder.map((columnId) => {
            const column = state.entities.columns[columnId];
            const tasks = column.taskIds.map(
            (taskId) => state.entities.tasks[taskId - 1]
          );

          return (
            <Column
              key={column.id}
              column={column}
              tasks={tasks}
              selectedTaskIds={state.selectedTaskIds}
              draggingTaskId={state.draggingTaskId}
              toggleSelection={toggleSelection}
              toggleSelectionInGroup={toggleSelectionInGroup}
              multiSelectTo={multiSelectTo}
            />
          );
        })}
      </DragDropContext>
    )
}

export default App;
