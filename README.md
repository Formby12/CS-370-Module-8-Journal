# CS-370-Module-8-Journal

Briefly explain the work that you did on this project: What code were you given? 
I was given two python classes that represent the environment, a maze defined as a matrix, and one that stores the episodes.
What code did you create yourself?
def qtrain(model, maze, **opt):

    # exploration factor
    global epsilon 

    # number of epochs
    n_epoch = opt.get('n_epoch', 15000)

    # maximum memory to store episodes
    max_memory = opt.get('max_memory', 1000)

    # maximum data size for training
    data_size = opt.get('data_size', 50)

    # start time
    start_time = datetime.datetime.now()

    # Construct environment/game from numpy array: maze (see above)
    qmaze = TreasureMaze(maze)

    # Initialize experience replay object
    experience = GameExperience(model, max_memory=max_memory)
    
    win_history = []   # history of win/lose game
    hsize = qmaze.maze.size//2   # history window size
    win_rate = 0.0
    
    # pseudocode:
    # For each epoch:
    for epoch in range(n_epoch):
        #    Agent_cell = randomly select a free cell
        agent_cell = random.choice(qmaze.free_cells) 
        
        #    Reset the maze with agent set to above position
        #    Hint: Review the reset method in the TreasureMaze.py class.
        qmaze.reset(agent_cell)
        
        #    envstate = Environment.current_state
        #    Hint: Review the observe method in the TreasureMaze.py class.
        envstate = qmaze.observe()
        
        n_episodes = 0
        loss = 0
        game_over = False
        
        #    While state is not game over:
        while qmaze.game_status() == 'not_over':
            
            # previous_envstate = envstate
            previous_envstate = envstate
        
            #        Action = randomly choose action (left, right, up, down) either by exploration or by exploitation
            if np.random.rand() < epsilon:
                action = random.choice(qmaze.valid_actions())
            else:
                q_values = model.predict(previous_envstate)[0]
                action = np.argmax(q_values) # Exploitation
            
            #        envstate, reward, game_status = qmaze.act(action)
            #    Hint: Review the act method in the TreasureMaze.py class.
            envstate, reward, game_status = qmaze.act(action)
            if game_status == 'win':
                win_history.append(1)
            elif game_status == 'lose':
                win_history.append(0)

            #        episode = [previous_envstate, action, reward, envstate, game_status]
            #        Store episode in Experience replay object
            #    Hint: Review the remember method in the GameExperience.py class.
            episode = [previous_envstate, action, reward, envstate, game_status == "win"]
            n_episodes += 1
            experience.remember(episode)
        
            #        Train neural network model and evaluate loss
            #    Hint: Call GameExperience.get_data to retrieve training data (input and target) and pass to model.fit method 
            #          to train the model. You can call model.evaluate to determine loss.
            inputs, targets = experience.get_data(data_size)
            model.fit(inputs, targets, epochs=40, batch_size=64, verbose=0)
            loss+= model.train_on_batch(inputs,targets)
           
            
            
            
            #    If the win rate is above the threshold and your model passes the completion check, that would be your epoch.
        win_rate = sum(win_history) / len(win_history)

    #Print the epoch, loss, episodes, win count, and win rate for each epoch
        dt = datetime.datetime.now() - start_time
        t = format_time(dt.total_seconds())
        template = "Epoch: {:03d}/{:d} | Loss: {:.4f} | Episodes: {:d} | Win count: {:d} | Win rate: {:.3f} | time: {}"
        print(template.format(epoch, n_epoch-1, loss, n_episodes, sum(win_history), win_rate, t))
        # We simply check if training has exhausted all free cells and if in all
        # cases the agent won.
        if win_rate > 0.9 : epsilon = 0.05
        if sum(win_history[-hsize:]) == hsize and completion_check(model, qmaze):
            print("Reached 100%% win rate at epoch: %d" % (epoch,))
            break
    
    
    # Determine the total time for training
    dt = datetime.datetime.now() - start_time
    seconds = dt.total_seconds()
    t = format_time(seconds)

    print("n_epoch: %d, max_mem: %d, data: %d, time: %s" % (epoch, max_memory, data_size, t))
    return seconds

# This is a small utility for printing readable time strings:
def format_time(seconds):
    if seconds < 400:
        s = float(seconds)
        return "%.1f seconds" % (s,)
    elif seconds < 4000:
        m = seconds / 60.0
        return "%.2f minutes" % (m,)
    else:
        h = seconds / 3600.0
        return "%.2f hours" % (h,)
Connect your learning from throughout this course to the larger field of computer science:
What do computer scientists do and why does it matter?
Computer scientists develop, improve, and design the software and hardware to help solve problems in many fields.
It matters because, it is a vital part of our digital world and it is going to help brighten the future of
technology
How do I approach a problem as a computer scientist?
I approach a problem by first understanding the problem, then planning a solution, write pseudocode, implement 
the solution, and test.
What are my ethical responsibilities to the end user and the organization?
The ethical responsibilities are to ensure user data is collected, store, and processed sercurely, Preventing hacking attempts 
or fraud, or allowing identity theft, to be non bias and transparent.
the  repsponsibilites for the organization is integrity and honesty, confidentiality, compliance, and ethics
