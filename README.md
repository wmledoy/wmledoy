<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Life RPG Planner</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background-color: #f5f5f5;
      color: #333;
    }

    header {
      background-color: #6200ea;
      padding: 20px;
      text-align: center;
      color: white;
    }

    nav {
      display: flex;
      justify-content: space-around;
      background-color: #3700b3;
      padding: 10px;
    }

    nav a {
      color: white;
      text-decoration: none;
      padding: 10px;
      border-radius: 5px;
    }

    nav a:hover {
      background-color: #bb86fc;
    }

    section {
      padding: 20px;
      display: none;
    }

    section.active {
      display: block;
    }

    .task-list, .notes-section {
      margin-top: 20px;
    }

    .task {
      margin-bottom: 10px;
      padding: 10px;
      background-color: #fff;
      border: 1px solid #ddd;
      border-radius: 5px;
    }

    .task.completed {
      text-decoration: line-through;
      color: gray;
    }

    .note {
      background: #fff;
      padding: 10px;
      border: 1px solid #ddd;
      margin-bottom: 10px;
      border-radius: 5px;
    }

    .add-section {
      margin-top: 20px;
      padding: 10px;
      background: #f5f5f5;
      border: 1px solid #ddd;
      border-radius: 5px;
    }

    .image-preview {
      max-width: 100%;
      max-height: 200px;
      margin-top: 10px;
    }

    /* Level bar */
    .level-bar {
      position: fixed;
      top: 20px;
      right: 20px;
      background-color: #333;
      color: white;
      padding: 10px;
      border-radius: 5px;
      font-size: 16px;
    }

    /* Pomodoro Timer */
    #pomodoro-timer {
      font-size: 48px;
      font-weight: bold;
    }

  </style>
</head>
<body>

  <header>
    <h1>Life RPG Planner</h1>
  </header>

  <!-- Level Indicator -->
  <div class="level-bar" id="level-bar">
    Level: 1 | XP: 0 / 100
  </div>

  <nav>
    <a href="#" onclick="switchSection('tasks')">Tasks</a>
    <a href="#" onclick="switchSection('calendar')">Calendar</a>
    <a href="#" onclick="switchSection('pomodoro')">Pomodoro</a>
    <a href="#" onclick="switchSection('notes')">Notes</a>
    <a href="#" onclick="switchSection('agenda')">Agenda</a>
  </nav>

  <!-- Sections -->
  <section id="tasks" class="active">
    <h2>Tasks</h2>
    <div class="task-list" id="task-list">
      <!-- Tasks will be rendered here -->
    </div>
    <div class="add-section">
      <h3>Add Task</h3>
      <input type="text" id="task-text" placeholder="Task description">
      <input type="date" id="task-date">
      <textarea id="task-details" placeholder="Details..."></textarea>
      <input type="file" id="task-image" accept="image/*">
      <button onclick="addTask()">Add</button>
    </div>
  </section>

  <section id="calendar">
    <h2>Calendar</h2>
    <input type="date" id="calendar-date" onchange="showTasksForDate()">
    <div id="tasks-for-date">
      <!-- Tasks for the selected date -->
    </div>
  </section>

  <section id="pomodoro">
    <h2>Pomodoro Timer</h2>
    <div id="pomodoro-timer">25:00</div>
    <button onclick="startPomodoro()">Start</button>
    <button onclick="stopPomodoro()">Stop</button>
    <button onclick="resetPomodoro()">Reset</button>
  </section>

  <section id="notes">
    <h2>Notes</h2>
    <div class="notes-section" id="notes-section">
      <!-- Notes will appear here -->
    </div>
    <div class="add-section">
      <h3>Add Note</h3>
      <textarea id="note-text" placeholder="Write your note..."></textarea>
      <input type="file" id="note-image" accept="image/*">
      <button onclick="addNote()">Add Note</button>
    </div>
  </section>

  <section id="agenda">
    <h2>Agenda</h2>
    <textarea id="agenda-text" rows="10" cols="50" placeholder="Write your agenda..."></textarea>
    <button onclick="saveAgenda()">Save Agenda</button>
  </section>

  <script>
    let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
    let notes = JSON.parse(localStorage.getItem('notes')) || [];
    let xp = 0; 
    let level = 1;
    let pomodoroInterval;
    let pomodoroTime = 1500; // 25 minutes
    
    function switchSection(sectionId) {
      document.querySelectorAll('section').forEach(section => {
        section.classList.remove('active');
      });
      document.getElementById(sectionId).classList.add('active');
    }

    function addTask() {
      const text = document.getElementById('task-text').value;
      const date = document.getElementById('task-date').value;
      const details = document.getElementById('task-details').value;
      const imageFile = document.getElementById('task-image').files[0];
      let imageUrl = '';

      if (imageFile) {
        const reader = new FileReader();
        reader.onload = () => {
          imageUrl = reader.result;
          tasks.push({ text, date, details, image: imageUrl, completed: false });
          localStorage.setItem('tasks', JSON.stringify(tasks));
          renderTasks();
        };
        reader.readAsDataURL(imageFile);
      } else {
        tasks.push({ text, date, details, image: imageUrl, completed: false });
        localStorage.setItem('tasks', JSON.stringify(tasks));
        renderTasks();
      }
    }

    function renderTasks() {
      const taskList = document.getElementById('task-list');
      taskList.innerHTML = '';

      tasks.forEach((task, index) => {
        const taskDiv = document.createElement('div');
        taskDiv.className = `task ${task.completed ? 'completed' : ''}`;
        taskDiv.innerHTML = `
          <p>${task.text}</p>
          <p>Due: ${task.date}</p>
          <p>${task.details}</p>
          ${task.image ? `<img src="${task.image}" class="image-preview">` : ''}
          <button onclick="toggleTask(${index})">${task.completed ? 'Undo' : 'Complete'}</button>
        `;
        taskList.appendChild(taskDiv);
      });
    }

    function toggleTask(index) {
      tasks[index].completed = !tasks[index].completed;
      localStorage.setItem('tasks', JSON.stringify(tasks));
      renderTasks();
    }

    function addNote() {
      const text = document.getElementById('note-text').value;
      const imageFile = document.getElementById('note-image').files[0];
      let imageUrl = '';

      if (imageFile) {
        const reader = new FileReader();
        reader.onload = () => {
          imageUrl = reader.result;
          notes.push({ text, image: imageUrl });
          localStorage.setItem('notes', JSON.stringify(notes));
          renderNotes();
        };
        reader.readAsDataURL(imageFile);
      } else {
        notes.push({ text, image: imageUrl });
        localStorage.setItem('notes', JSON.stringify(notes));
        renderNotes();
      }
    }

    function renderNotes() {
      const notesSection = document.getElementById('notes-section');
      notesSection.innerHTML = '';

      notes.forEach(note => {
        const noteDiv = document.createElement('div');
        noteDiv.className = 'note';
        noteDiv.innerHTML = `
          <p>${note.text}</p>
          ${note.image ? `<img src="${note.image}" class="image-preview">` : ''}
          <button onclick="deleteNote(${notes.indexOf(note)})">Delete</button>
        `;
        notesSection.appendChild(noteDiv);
      });
    }

    function deleteNote(index) {
      notes.splice(index, 1);
      localStorage.setItem('notes', JSON.stringify(notes));
      renderNotes();
    }

    function saveAgenda() {
      const agendaText = document.getElementById('agenda-text').value;
      localStorage.setItem('agenda', agendaText);
      alert('Agenda saved!');
    }

    document.addEventListener('DOMContentLoaded', () => {
      renderTasks();
      renderNotes();
      document.getElementById('agenda-text').value = localStorage.getItem('agenda') || '';
    });

    // Pomodoro Timer
    function startPomodoro() {
      pomodoroInterval = setInterval(() => {
        if (pomodoroTime > 0) {
          pomodoroTime--;
          const minutes = Math.floor(pomodoroTime / 60);
          const seconds = pomodoroTime % 60;
          document.getElementById('pomodoro-timer').textContent = `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
        } else {
          clearInterval(pomodoroInterval);
          alert('Pomodoro session complete!');
        }
      }, 1000);
    }

    function stopPomodoro() {
      clearInterval(pomodoroInterval);
    }

    function resetPomodoro() {
      clearInterval(pomodoroInterval);
      pomodoroTime = 1500; // Reset to 25 minutes
      document.getElementById('pomodoro-timer').textContent = '25:00';
    }
  </script>

</body>
</html><!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Life RPG Planner</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background-color: #f5f5f5;
      color: #333;
    }

    header {
      background-color: #6200ea;
      padding: 20px;
      text-align: center;
      color: white;
    }

    nav {
      display: flex;
      justify-content: space-around;
      background-color: #3700b3;
      padding: 10px;
    }

    nav a {
      color: white;
      text-decoration: none;
      padding: 10px;
      border-radius: 5px;
    }

    nav a:hover {
      background-color: #bb86fc;
    }

    section {
      padding: 20px;
      display: none;
    }

    section.active {
      display: block;
    }

    .task-list, .notes-section {
      margin-top: 20px;
    }

    .task {
      margin-bottom: 10px;
      padding: 10px;
      background-color: #fff;
      border: 1px solid #ddd;
      border-radius: 5px;
    }

    .task.completed {
      text-decoration: line-through;
      color: gray;
    }

    .note {
      background: #fff;
      padding: 10px;
      border: 1px solid #ddd;
      margin-bottom: 10px;
      border-radius: 5px;
    }

    .add-section {
      margin-top: 20px;
      padding: 10px;
      background: #f5f5f5;
      border: 1px solid #ddd;
      border-radius: 5px;
    }

    .image-preview {
      max-width: 100%;
      max-height: 200px;
      margin-top: 10px;
    }

    /* Level bar */
    .level-bar {
      position: fixed;
      top: 20px;
      right: 20px;
      background-color: #333;
      color: white;
      padding: 10px;
      border-radius: 5px;
      font-size: 16px;
    }

    /* Pomodoro Timer */
    #pomodoro-timer {
      font-size: 48px;
      font-weight: bold;
    }

  </style>
</head>
<body>

  <header>
    <h1>Life RPG Planner</h1>
  </header>

  <!-- Level Indicator -->
  <div class="level-bar" id="level-bar">
    Level: 1 | XP: 0 / 100
  </div>

  <nav>
    <a href="#" onclick="switchSection('tasks')">Tasks</a>
    <a href="#" onclick="switchSection('calendar')">Calendar</a>
    <a href="#" onclick="switchSection('pomodoro')">Pomodoro</a>
    <a href="#" onclick="switchSection('notes')">Notes</a>
    <a href="#" onclick="switchSection('agenda')">Agenda</a>
  </nav>

  <!-- Sections -->
  <section id="tasks" class="active">
    <h2>Tasks</h2>
    <div class="task-list" id="task-list">
      <!-- Tasks will be rendered here -->
    </div>
    <div class="add-section">
      <h3>Add Task</h3>
      <input type="text" id="task-text" placeholder="Task description">
      <input type="date" id="task-date">
      <textarea id="task-details" placeholder="Details..."></textarea>
      <input type="file" id="task-image" accept="image/*">
      <button onclick="addTask()">Add</button>
    </div>
  </section>

  <section id="calendar">
    <h2>Calendar</h2>
    <input type="date" id="calendar-date" onchange="showTasksForDate()">
    <div id="tasks-for-date">
      <!-- Tasks for the selected date -->
    </div>
  </section>

  <section id="pomodoro">
    <h2>Pomodoro Timer</h2>
    <div id="pomodoro-timer">25:00</div>
    <button onclick="startPomodoro()">Start</button>
    <button onclick="stopPomodoro()">Stop</button>
    <button onclick="resetPomodoro()">Reset</button>
  </section>

  <section id="notes">
    <h2>Notes</h2>
    <div class="notes-section" id="notes-section">
      <!-- Notes will appear here -->
    </div>
    <div class="add-section">
      <h3>Add Note</h3>
      <textarea id="note-text" placeholder="Write your note..."></textarea>
      <input type="file" id="note-image" accept="image/*">
      <button onclick="addNote()">Add Note</button>
    </div>
  </section>

  <section id="agenda">
    <h2>Agenda</h2>
    <textarea id="agenda-text" rows="10" cols="50" placeholder="Write your agenda..."></textarea>
    <button onclick="saveAgenda()">Save Agenda</button>
  </section>

  <script>
    let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
    let notes = JSON.parse(localStorage.getItem('notes')) || [];
    let xp = 0; 
    let level = 1;
    let pomodoroInterval;
    let pomodoroTime = 1500; // 25 minutes
    
    function switchSection(sectionId) {
      document.querySelectorAll('section').forEach(section => {
        section.classList.remove('active');
      });
      document.getElementById(sectionId).classList.add('active');
    }

    function addTask() {
      const text = document.getElementById('task-text').value;
      const date = document.getElementById('task-date').value;
      const details = document.getElementById('task-details').value;
      const imageFile = document.getElementById('task-image').files[0];
      let imageUrl = '';

      if (imageFile) {
        const reader = new FileReader();
        reader.onload = () => {
          imageUrl = reader.result;
          tasks.push({ text, date, details, image: imageUrl, completed: false });
          localStorage.setItem('tasks', JSON.stringify(tasks));
          renderTasks();
        };
        reader.readAsDataURL(imageFile);
      } else {
        tasks.push({ text, date, details, image: imageUrl, completed: false });
        localStorage.setItem('tasks', JSON.stringify(tasks));
        renderTasks();
      }
    }

    function renderTasks() {
      const taskList = document.getElementById('task-list');
      taskList.innerHTML = '';

      tasks.forEach((task, index) => {
        const taskDiv = document.createElement('div');
        taskDiv.className = `task ${task.completed ? 'completed' : ''}`;
        taskDiv.innerHTML = `
          <p>${task.text}</p>
          <p>Due: ${task.date}</p>
          <p>${task.details}</p>
          ${task.image ? `<img src="${task.image}" class="image-preview">` : ''}
          <button onclick="toggleTask(${index})">${task.completed ? 'Undo' : 'Complete'}</button>
        `;
        taskList.appendChild(taskDiv);
      });
    }

    function toggleTask(index) {
      tasks[index].completed = !tasks[index].completed;
      localStorage.setItem('tasks', JSON.stringify(tasks));
      renderTasks();
    }

    function addNote() {
      const text = document.getElementById('note-text').value;
      const imageFile = document.getElementById('note-image').files[0];
      let imageUrl = '';

      if (imageFile) {
        const reader = new FileReader();
        reader.onload = () => {
          imageUrl = reader.result;
          notes.push({ text, image: imageUrl });
          localStorage.setItem('notes', JSON.stringify(notes));
          renderNotes();
        };
        reader.readAsDataURL(imageFile);
      } else {
        notes.push({ text, image: imageUrl });
        localStorage.setItem('notes', JSON.stringify(notes));
        renderNotes();
      }
    }

    function renderNotes() {
      const notesSection = document.getElementById('notes-section');
      notesSection.innerHTML = '';

      notes.forEach(note => {
        const noteDiv = document.createElement('div');
        noteDiv.className = 'note';
        noteDiv.innerHTML = `
          <p>${note.text}</p>
          ${note.image ? `<img src="${note.image}" class="image-preview">` : ''}
          <button onclick="deleteNote(${notes.indexOf(note)})">Delete</button>
        `;
        notesSection.appendChild(noteDiv);
      });
    }

    function deleteNote(index) {
      notes.splice(index, 1);
      localStorage.setItem('notes', JSON.stringify(notes));
      renderNotes();
    }

    function saveAgenda() {
      const agendaText = document.getElementById('agenda-text').value;
      localStorage.setItem('agenda', agendaText);
      alert('Agenda saved!');
    }

    document.addEventListener('DOMContentLoaded', () => {
      renderTasks();
      renderNotes();
      document.getElementById('agenda-text').value = localStorage.getItem('agenda') || '';
    });

    // Pomodoro Timer
    function startPomodoro() {
      pomodoroInterval = setInterval(() => {
        if (pomodoroTime > 0) {
          pomodoroTime--;
          const minutes = Math.floor(pomodoroTime / 60);
          const seconds = pomodoroTime % 60;
          document.getElementById('pomodoro-timer').textContent = `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
        } else {
          clearInterval(pomodoroInterval);
          alert('Pomodoro session complete!');
        }
      }, 1000);
    }

    function stopPomodoro() {
      clearInterval(pomodoroInterval);
    }

    function resetPomodoro() {
      clearInterval(pomodoroInterval);
      pomodoroTime = 1500; // Reset to 25 minutes
      document.getElementById('pomodoro-timer').textContent = '25:00';
    }
  </script>

</body>
</html>
