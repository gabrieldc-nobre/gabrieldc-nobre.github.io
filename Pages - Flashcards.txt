import React, { useState, useMemo } from 'react';
import { base44 } from '@/api/base44Client';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { Search, Filter, ListTodo, CheckCircle2, Clock, AlertCircle, TrendingUp, SlidersHorizontal } from 'lucide-react';
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { motion } from 'framer-motion';

import FlashCard from '../components/flashcards/FlashCard';
import StatsCard from '../components/common/StatsCard';
import EditableTitle from '../components/common/EditableTitle';
import LogoUploader from '../components/common/LogoUploader';
import TaskModal from '../components/gantt/TaskModal';

export default function Flashcards() {
  const queryClient = useQueryClient();
  const [searchTerm, setSearchTerm] = useState('');
  const [categoryFilter, setCategoryFilter] = useState('all');
  const [responsibleFilter, setResponsibleFilter] = useState('all');
  const [statusFilter, setStatusFilter] = useState('all');
  const [showTaskModal, setShowTaskModal] = useState(false);
  const [selectedTask, setSelectedTask] = useState(null);
  const [showFilters, setShowFilters] = useState(false);

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
    const completed = tasks.filter(t => t.progresso === 100).length;
    const inProgress = tasks.filter(t => t.progresso > 0 && t.progresso < 100).length;
    const notStarted = tasks.filter(t => t.progresso === 0).length;
    const overallProgress = total > 0 
      ? Math.round(tasks.reduce((acc, t) => acc + t.progresso, 0) / total) 
      : 0;

    return { total, completed, inProgress, notStarted, overallProgress };
  }, [tasks]);

  // Filtered tasks
  const filteredTasks = useMemo(() => {
    return tasks.filter(task => {
      const matchesSearch = task.tarefa.toLowerCase().includes(searchTerm.toLowerCase()) ||
                           task.categoria.toLowerCase().includes(searchTerm.toLowerCase());
      const matchesCategory = categoryFilter === 'all' || task.categoria === categoryFilter;
      const matchesResponsible = responsibleFilter === 'all' || task.responsavel === responsibleFilter;
      
      let matchesStatus = true;
      if (statusFilter === 'completed') matchesStatus = task.progresso === 100;
      else if (statusFilter === 'inProgress') matchesStatus = task.progresso > 0 && task.progresso < 100;
      else if (statusFilter === 'notStarted') matchesStatus = task.progresso === 0;

      return matchesSearch && matchesCategory && matchesResponsible && matchesStatus;
    });
  }, [tasks, searchTerm, categoryFilter, responsibleFilter, statusFilter]);

  const handleTaskClick = (task) => {
    setSelectedTask(task);
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
      <div className="max-w-[1600px] mx-auto">
        {/* Header */}
        <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-6">
          <div className="flex items-center gap-4">
            <LogoUploader 
              logoUrl={settings?.logo_url} 
              onLogoChange={(url) => updateSettings.mutate({ logo_url: url })}
            />
            <EditableTitle 
              value={settings?.titulo_flashcards || 'Visão Geral Gerencial'} 
              onChange={(value) => updateSettings.mutate({ titulo_flashcards: value })}
            />
          </div>
        </div>

        {/* Stats Cards */}
        <div className="grid grid-cols-2 md:grid-cols-5 gap-4 mb-6">
          <StatsCard
            title="Total de Tarefas"
            value={stats.total}
            icon={ListTodo}
            color="indigo"
            onClick={() => setStatusFilter('all')}
            active={statusFilter === 'all'}
          />
          <StatsCard
            title="Concluídas"
            value={stats.completed}
            icon={CheckCircle2}
            color="emerald"
            onClick={() => setStatusFilter('completed')}
            active={statusFilter === 'completed'}
          />
          <StatsCard
            title="Em Andamento"
            value={stats.inProgress}
            icon={Clock}
            color="amber"
            onClick={() => setStatusFilter('inProgress')}
            active={statusFilter === 'inProgress'}
          />
          <StatsCard
            title="Não Iniciadas"
            value={stats.notStarted}
            icon={AlertCircle}
            color="slate"
            onClick={() => setStatusFilter('notStarted')}
            active={statusFilter === 'notStarted'}
          />
          <StatsCard
            title="Progresso Geral"
            value={`${stats.overallProgress}%`}
            icon={TrendingUp}
            color="rose"
          />
        </div>

        {/* Search & Filters */}
        <div className="bg-white rounded-2xl shadow-sm border border-slate-100 p-4 mb-6">
          <div className="flex flex-col md:flex-row gap-4">
            <div className="flex-1 relative">
              <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-slate-400" />
              <Input
                placeholder="Buscar tarefas..."
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
                className="pl-10 h-11 rounded-xl border-slate-200"
              />
            </div>
            <Button
              variant="outline"
              onClick={() => setShowFilters(!showFilters)}
              className="rounded-xl gap-2 h-11 border-slate-200"
            >
              <SlidersHorizontal className="h-4 w-4" />
              Filtros
            </Button>
          </div>

          {showFilters && (
            <motion.div
              initial={{ opacity: 0, height: 0 }}
              animate={{ opacity: 1, height: 'auto' }}
              exit={{ opacity: 0, height: 0 }}
              className="grid grid-cols-1 md:grid-cols-3 gap-4 mt-4 pt-4 border-t border-slate-100"
            >
              <Select value={categoryFilter} onValueChange={setCategoryFilter}>
                <SelectTrigger className="h-11 rounded-xl border-slate-200">
                  <SelectValue placeholder="Categoria" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="all">Todas Categorias</SelectItem>
                  {categories.map((cat) => (
                    <SelectItem key={cat.id} value={cat.nome}>{cat.nome}</SelectItem>
                  ))}
                </SelectContent>
              </Select>

              <Select value={responsibleFilter} onValueChange={setResponsibleFilter}>
                <SelectTrigger className="h-11 rounded-xl border-slate-200">
                  <SelectValue placeholder="Responsável" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="all">Todos Responsáveis</SelectItem>
                  {responsibles.map((resp) => (
                    <SelectItem key={resp.id} value={resp.nome}>{resp.nome}</SelectItem>
                  ))}
                </SelectContent>
              </Select>

              <Select value={statusFilter} onValueChange={setStatusFilter}>
                <SelectTrigger className="h-11 rounded-xl border-slate-200">
                  <SelectValue placeholder="Status" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="all">Todos Status</SelectItem>
                  <SelectItem value="completed">Concluídas</SelectItem>
                  <SelectItem value="inProgress">Em Andamento</SelectItem>
                  <SelectItem value="notStarted">Não Iniciadas</SelectItem>
                </SelectContent>
              </Select>
            </motion.div>
          )}
        </div>

        {/* Flashcards Grid */}
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
          {filteredTasks.map((task, index) => (
            <FlashCard
              key={task.id}
              task={task}
              onClick={() => handleTaskClick(task)}
              index={index}
            />
          ))}
        </div>

        {filteredTasks.length === 0 && (
          <div className="text-center py-16">
            <p className="text-slate-500">Nenhuma tarefa encontrada com os filtros selecionados.</p>
          </div>
        )}

        {/* Task Modal */}
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