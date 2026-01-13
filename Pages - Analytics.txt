import React, { useState, useMemo } from 'react';
import { base44 } from '@/api/base44Client';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { addMonths, subMonths } from 'date-fns';
import { ChevronLeft, ChevronRight, ListTodo, CheckCircle2, Clock, AlertCircle, TrendingUp } from 'lucide-react';
import { Button } from "@/components/ui/button";
import { format } from 'date-fns';
import { ptBR } from 'date-fns/locale';
import { motion } from 'framer-motion';

import TaskCharts from '../components/charts/TaskCharts';
import StatsCard from '../components/common/StatsCard';
import EditableTitle from '../components/common/EditableTitle';
import LogoUploader from '../components/common/LogoUploader';
import TaskListModal from '../components/common/TaskListModal';
import TaskModal from '../components/gantt/TaskModal';

export default function Analytics() {
  const queryClient = useQueryClient();
  const [currentDate, setCurrentDate] = useState(new Date(2026, 0, 1));
  const [selectedFilter, setSelectedFilter] = useState(null);
  const [showTaskListModal, setShowTaskListModal] = useState(false);
  const [showTaskModal, setShowTaskModal] = useState(false);
  const [selectedTask, setSelectedTask] = useState(null);

  const { data: tasks = [] } = useQuery({
    queryKey: ['tasks'],
    queryFn: () => base44.entities.Task.list()
  });

  const { data: categories = [] } = useQuery({
    queryKey: ['categories'],
    queryFn: () => base44.entities.Category.list()
  });

  const { data: responsibles = [] } = useQuery({
    queryKey: ['responsibles'],
    queryFn: () => base44.entities.Responsible.list()
  });

  const { data: settings } = useQuery({
    queryKey: ['settings'],
    queryFn: async () => {
      const list = await base44.entities.AppSettings.list();
      return list[0] || null;
    }
  });

  const updateSettings = useMutation({
    mutationFn: async (data) => {
      if (settings?.id) {
        return base44.entities.AppSettings.update(settings.id, data);
      } else {
        return base44.entities.AppSettings.create(data);
      }
    },
    onSuccess: () => queryClient.invalidateQueries(['settings'])
  });

  const updateTask = useMutation({
    mutationFn: ({ id, data }) => base44.entities.Task.update(id, data),
    onSuccess: () => queryClient.invalidateQueries(['tasks'])
  });

  const deleteTask = useMutation({
    mutationFn: (id) => base44.entities.Task.delete(id),
    onSuccess: () => queryClient.invalidateQueries(['tasks'])
  });

  // Stats
  const stats = useMemo(() => {
    const total = tasks.length;
    const completed = tasks.filter(t => t.progresso === 100);
    const inProgress = tasks.filter(t => t.progresso > 0 && t.progresso < 100);
    const notStarted = tasks.filter(t => t.progresso === 0);
    const overallProgress = total > 0 
      ? Math.round(tasks.reduce((acc, t) => acc + t.progresso, 0) / total) 
      : 0;

    return {
      total,
      completed,
      inProgress,
      notStarted,
      overallProgress
    };
  }, [tasks]);

  const handleFilterClick = (filter) => {
    setSelectedFilter(filter);
    setShowTaskListModal(true);
  };

  const getFilteredTasks = () => {
    if (!selectedFilter) return [];
    switch (selectedFilter) {
      case 'total': return tasks;
      case 'completed': return stats.completed;
      case 'inProgress': return stats.inProgress;
      case 'notStarted': return stats.notStarted;
      default: return [];
    }
  };

  const getFilterTitle = () => {
    switch (selectedFilter) {
      case 'total': return 'Todas as Tarefas';
      case 'completed': return 'Tarefas Concluídas';
      case 'inProgress': return 'Tarefas em Andamento';
      case 'notStarted': return 'Tarefas Não Iniciadas';
      default: return '';
    }
  };

  const handleTaskClick = (task) => {
    setSelectedTask(task);
    setShowTaskListModal(false);
    setShowTaskModal(true);
  };

  const handleSaveTask = (data, taskId) => {
    if (taskId) {
      updateTask.mutate({ id: taskId, data });
    }
    setShowTaskModal(false);
    setSelectedTask(null);
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-50 via-white to-indigo-50/30 p-4 md:p-8">
      <div className="max-w-[1400px] mx-auto">
        {/* Header */}
        <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-8">
          <div className="flex items-center gap-4">
            <LogoUploader 
              logoUrl={settings?.logo_url} 
              onLogoChange={(url) => updateSettings.mutate({ logo_url: url })}
            />
            <EditableTitle 
              value={settings?.titulo_graficos || 'Analytics do Projeto'} 
              onChange={(value) => updateSettings.mutate({ titulo_graficos: value })}
            />
          </div>
          
          <div className="flex items-center gap-3 bg-white rounded-xl p-1 shadow-sm border border-slate-100">
            <Button 
              variant="ghost" 
              size="icon" 
              onClick={() => setCurrentDate(prev => subMonths(prev, 1))}
              className="h-9 w-9 rounded-lg"
            >
              <ChevronLeft className="h-5 w-5" />
            </Button>
            <span className="text-sm font-medium text-slate-700 min-w-[140px] text-center capitalize">
              {format(currentDate, 'MMMM yyyy', { locale: ptBR })}
            </span>
            <Button 
              variant="ghost" 
              size="icon" 
              onClick={() => setCurrentDate(prev => addMonths(prev, 1))}
              className="h-9 w-9 rounded-lg"
            >
              <ChevronRight className="h-5 w-5" />
            </Button>
          </div>
        </div>

        {/* Stats Cards */}
        <div className="grid grid-cols-2 md:grid-cols-5 gap-4 mb-8">
          <StatsCard
            title="Total de Tarefas"
            value={stats.total}
            icon={ListTodo}
            color="indigo"
            onClick={() => handleFilterClick('total')}
            active={selectedFilter === 'total'}
          />
          <StatsCard
            title="Concluídas"
            value={stats.completed.length}
            icon={CheckCircle2}
            color="emerald"
            onClick={() => handleFilterClick('completed')}
            active={selectedFilter === 'completed'}
          />
          <StatsCard
            title="Em Andamento"
            value={stats.inProgress.length}
            icon={Clock}
            color="amber"
            onClick={() => handleFilterClick('inProgress')}
            active={selectedFilter === 'inProgress'}
          />
          <StatsCard
            title="Não Iniciadas"
            value={stats.notStarted.length}
            icon={AlertCircle}
            color="slate"
            onClick={() => handleFilterClick('notStarted')}
            active={selectedFilter === 'notStarted'}
          />
          <StatsCard
            title="Progresso Geral"
            value={`${stats.overallProgress}%`}
            icon={TrendingUp}
            color="rose"
          />
        </div>

        {/* Charts */}
        <TaskCharts tasks={tasks} currentDate={currentDate} />

        {/* Task List Modal */}
        <TaskListModal
          open={showTaskListModal}
          onClose={() => setShowTaskListModal(false)}
          title={getFilterTitle()}
          tasks={getFilteredTasks()}
          onTaskClick={handleTaskClick}
        />

        {/* Task Edit Modal */}
        <TaskModal
          open={showTaskModal}
          onClose={() => setShowTaskModal(false)}
          task={selectedTask}
          categories={categories}
          responsibles={responsibles}
          onSave={handleSaveTask}
          onDelete={(id) => {
            deleteTask.mutate(id);
            setShowTaskModal(false);
            setSelectedTask(null);
          }}
        />
      </div>
    </div>
  );
}