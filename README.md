import math

def solve():
    # Чтение n и d
    try:
        n_str = input()
        if not n_str:
            return
        n = int(n_str)
        d = float(input()) # Расстояние d может быть целым, но работаем с float
    except EOFError:
        return
    except ValueError:
        # Может быть, входные данные подаются все сразу
        pass
    
    # Чтение времен t_i
    T = []
    # Чтение времен A
    for _ in range(n):
        T.append(float(input()))
    # Чтение времен B
    for _ in range(n):
        T.append(float(input()))

    # Создаем полный список t_i, где t[i] это время старта лыжника i+1
    # T[0] = t1 (лыжник 1), T[1] = t2 (лыжник 2), ..., T[2n-1] = t2n (лыжник 2n)
    
    # Инициализация
    current_runner_idx = 1 # Индекс лыжника, несущего флаг (1, 2, ..., 2n)
    start_time = T[0]      # Время, когда лыжник current_runner_idx начал движение с флагом
    start_pos = 0.0        # Позиция, с которой он начал
    total_distance = 0.0   # Общее пройденное расстояние флагом

    # Множество лыжников, которые уже несли флаг.
    # Лыжники могут быть встречены только 1 раз.
    carried_flag = {1}
    
    # Моделирование
    while True:
        # Команда A - нечетные индексы (1, 3, 5, ...), стартуют из 0, движутся к d (+1)
        # Команда B - четные индексы (2, 4, 6, ...), стартуют из d, движутся к 0 (-1)
        
        is_A = (current_runner_idx % 2 != 0) # Текущий лыжник из A?
        
        # 1. Рассчитать время доставки флага
        if is_A:
            # Движение к d (B)
            time_to_B = (d - start_pos) / 1.0
            delivery_time = start_time + time_to_B
            delivery_pos = d
        else:
            # Движение к 0 (A)
            time_to_A = (start_pos - 0.0) / 1.0
            delivery_time = start_time + time_to_A
            delivery_pos = 0.0

        min_meeting_time = delivery_time
        next_runner_idx = 0
        meeting_pos = delivery_pos
        
        # 2. Поиск ближайшей встречи
        # Противоположная команда: B (четные) если is_A, A (нечетные) если not is_A
        opponent_start_idx = 2 if is_A else 1
        
        for i in range(opponent_start_idx, 2 * n + 1, 2):
            if i in carried_flag:
                continue

            t_o = T[i - 1] # Время старта лыжника i
            
            # Расчет встречи (текущий C (start_pos, start_time) и оппонент O (t_o))
            
            # O (B -> A), C (A -> B)
            if is_A:
                # C (из A, +1): x_C(t) = start_pos + t - start_time
                # O (из B, -1): x_O(t) = d - (t - t_o)  (для t >= t_o)
                # 2*t = d + t_o + start_time - start_pos
                t_meet = (d + t_o + start_time - start_pos) / 2.0
                x_meet = start_pos + (t_meet - start_time)
            
            # O (A -> B), C (B -> A)
            else:
                # C (из B, -1): x_C(t) = start_pos - (t - start_time)
                # O (из A, +1): x_O(t) = t - t_o       (для t >= t_o)
                # 2*t = start_pos + start_time + t_o
                t_meet = (start_pos + start_time + t_o) / 2.0
                x_meet = start_pos - (t_meet - start_time)

            # Проверка, что встреча произошла раньше доставки, и оппонент успел стартовать
            # Также проверяем, что 0 <= x_meet <= d, хотя это должно быть гарантировано,
            # если t_meet < delivery_time.
            if t_meet < min_meeting_time and t_meet >= t_o:
                min_meeting_time = t_meet
                next_runner_idx = i
                meeting_pos = x_meet
        
        # 3. Принятие решения
        
        if next_runner_idx != 0:
            # Встреча
            distance_travelled = abs(min_meeting_time - start_time) * 1.0
            total_distance += distance_travelled
            
            # Обновление состояния для следующего этапа
            start_time = min_meeting_time
            start_pos = meeting_pos
            current_runner_idx = next_runner_idx
            carried_flag.add(next_runner_idx)
            
            # Замечание: если флаг принесен в пункт в момент старта, то это победа.
            # Если min_meeting_time == delivery_time, нужно проверить.
            # Однако, поскольку в случае встречи min_meeting_time < delivery_time,
            # это условие будет обрабатываться корректно.
            
        else:
            # Доставка флага (или победа)
            distance_travelled = abs(delivery_time - start_time) * 1.0
            total_distance += distance_travelled
            
            # Проверка условия победы: "если флаг принесут в один из пунктов в тот момент,
            # когда из этого пункта стартует очередной лыжник, — это считается победой,
            # эстафета окончена."
            # В примере: момент 13 - старт лыжника 8 из B, но лыжник 7 в этот момент прибежит
            # с флагом в пункт B, и команда A победит.
            
            # Ищем, стартует ли кто-то в delivery_time из delivery_pos.
            
            # Если лыжник C (из A) доставил флаг в B (d):
            if is_A:
                # Ищем лыжника B (четный), который стартует в B в delivery_time
                for i in range(2, 2 * n + 1, 2):
                    if abs(T[i - 1] - delivery_time) < 1e-9: # Сравнение float
                        # Если лыжник B стартует в B в этот момент, то A побеждает.
                        # Фактически, это уже победа, так как флаг доставлен.
                        pass
            
            # Если лыжник C (из B) доставил флаг в A (0):
            else:
                # Ищем лыжника A (нечетный), который стартует в A в delivery_time
                for i in range(1, 2 * n + 1, 2):
                    if abs(T[i - 1] - delivery_time) < 1e-9: # Сравнение float
                        # Если лыжник A стартует в A в этот момент, то B побеждает.
                        pass
            
            # В любом случае, доставка в пункт - это конец эстафеты.
            break

    # Ответ должен быть целым числом, округляем для надежности.
    # Возвращаем int(round(total_distance))
    print(f"{round(total_distance)}")

# Запуск функции для обработки ввода
# Поскольку ввод данных в этой среде является нетривиальным,
# и требуется точное решение для олимпиадной задачи,
# я ограничусь логическим алгоритмом и кодом, который должен

