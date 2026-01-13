import React, { useMemo } from 'react';
import { PieChart, Pie, Cell, LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, Legend, BarChart, Bar } from 'recharts';
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { parseISO, format, eachDayOfInterval, startOfMonth, endOfMonth, isWithinInterval } from 'date-fns';
import { ptBR } from 'date-fns/locale';

const COLORS = ['#6366f1', '#10b981', '#f59e0b', '#ef4444', '#8b5cf6', '#ec4899', '#06b6d4', '#84cc16'];

export default function TaskCharts({ tasks, currentDate }) {
  // Dados para gráfico de rosca - Status
  const statusData = useMemo(() => {
    const completed = tasks.filter(t => t.progresso === 100).length;
    const inProgress = tasks.filter(t => t.progresso > 0 && t.progresso < 100).length;
    const notStarted = tasks.filter(t => t.progresso === 0).length;
    
    return [
      { name: 'Concluídas', value: completed, color: '#10b981' },
      { name: 'Em Andamento', value: inProgress, color: '#f59e0b' },
      { name: 'Não Iniciadas', value: notStarted, color: '#94a3b8' },
    ].filter(d => d.value > 0);
  }, [tasks]);

  // Dados para gráfico de rosca - Por Responsável
  const responsibleData = useMemo(() => {
    const grouped = {};
    tasks.forEach(task => {
      if (!grouped[task.responsavel]) {
        grouped[task.responsavel] = 0;
      }
      grouped[task.responsavel]++;
    });
    return Object.entries(grouped).map(([name, value], idx) => ({
      name,
      value,
      color: COLORS[idx % COLORS.length]
    }));
  }, [tasks]);

  // Dados para gráfico de rosca - Por Categoria
  const categoryData = useMemo(() => {
    const grouped = {};
    tasks.forEach(task => {
      if (!grouped[task.categoria]) {
        grouped[task.categoria] = 0;
      }
      grouped[task.categoria]++;
    });
    return Object.entries(grouped).map(([name, value], idx) => ({
      name: name.length > 20 ? name.substring(0, 20) + '...' : name,
      fullName: name,
      value,
      color: COLORS[idx % COLORS.length]
    }));
  }, [tasks]);

  // Tarefas executadas por dia (considerando apenas tarefas com progresso > 0)
  const tasksPerDay = useMemo(() => {
    const monthStart = startOfMonth(currentDate);
    const monthEnd = endOfMonth(currentDate);
    const days = eachDayOfInterval({ start: monthStart, end: monthEnd });
    
    return days.map(day => {
      const dayStr = format(day, 'yyyy-MM-dd');
      // Conta tarefas que foram iniciadas (progresso > 0) até este dia
      const activeTasks = tasks.filter(task => {
        const taskStart = parseISO(task.inicio);
        const taskEnd = parseISO(task.termino);
        const isInRange = isWithinInterval(day, { start: taskStart, end: taskEnd });
        return isInRange && task.progresso > 0;
      }).length;
      
      return {
        date: format(day, 'dd', { locale: ptBR }),
        tarefas: activeTasks
      };
    });
  }, [tasks, currentDate]);

  // Progresso por responsável
  const progressByResponsible = useMemo(() => {
    const grouped = {};
    tasks.forEach(task => {
      if (!grouped[task.responsavel]) {
        grouped[task.responsavel] = { total: 0, progress: 0, count: 0 };
      }
      grouped[task.responsavel].progress += task.progresso;
      grouped[task.responsavel].count++;
    });
    
    return Object.entries(grouped).map(([name, data]) => ({
      name,
      progresso: Math.round(data.progress / data.count),
      tarefas: data.count
    }));
  }, [tasks]);

  const CustomTooltip = ({ active, payload, label }) => {
    if (active && payload && payload.length) {
      return (
        <div className="bg-white p-3 rounded-xl shadow-lg border border-slate-100">
          <p className="text-sm font-medium text-slate-800">{label || payload[0].name}</p>
          <p className="text-sm text-slate-600">
            {payload[0].value} {payload[0].name === 'progresso' ? '%' : 'tarefas'}
          </p>
        </div>
      );
    }
    return null;
  };

  return (
    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
      {/* Status das Tarefas */}
      <Card className="shadow-sm border-slate-100 rounded-2xl">
        <CardHeader className="pb-2">
          <CardTitle className="text-lg font-semibold text-slate-800">Status das Tarefas</CardTitle>
        </CardHeader>
        <CardContent>
          <ResponsiveContainer width="100%" height={250}>
            <PieChart>
              <Pie
                data={statusData}
                innerRadius={60}
                outerRadius={90}
                paddingAngle={5}
                dataKey="value"
              >
                {statusData.map((entry, index) => (
                  <Cell key={`cell-${index}`} fill={entry.color} />
                ))}
              </Pie>
              <Tooltip content={<CustomTooltip />} />
              <Legend 
                verticalAlign="bottom" 
                height={36}
                formatter={(value) => <span className="text-sm text-slate-600">{value}</span>}
              />
            </PieChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>

      {/* Tarefas por Responsável */}
      <Card className="shadow-sm border-slate-100 rounded-2xl">
        <CardHeader className="pb-2">
          <CardTitle className="text-lg font-semibold text-slate-800">Tarefas por Responsável</CardTitle>
        </CardHeader>
        <CardContent>
          <ResponsiveContainer width="100%" height={250}>
            <PieChart>
              <Pie
                data={responsibleData}
                innerRadius={60}
                outerRadius={90}
                paddingAngle={5}
                dataKey="value"
              >
                {responsibleData.map((entry, index) => (
                  <Cell key={`cell-${index}`} fill={entry.color} />
                ))}
              </Pie>
              <Tooltip content={<CustomTooltip />} />
              <Legend 
                verticalAlign="bottom" 
                height={36}
                formatter={(value) => <span className="text-sm text-slate-600">{value}</span>}
              />
            </PieChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>

      {/* Tarefas por Dia do Mês */}
      <Card className="shadow-sm border-slate-100 rounded-2xl lg:col-span-2">
        <CardHeader className="pb-2">
          <CardTitle className="text-lg font-semibold text-slate-800">
            Tarefas em Execução por Dia ({format(currentDate, 'MMMM yyyy', { locale: ptBR })})
          </CardTitle>
          <p className="text-sm text-slate-500">Apenas tarefas com progresso superior a 0%</p>
        </CardHeader>
        <CardContent>
          <ResponsiveContainer width="100%" height={280}>
            <LineChart data={tasksPerDay}>
              <CartesianGrid strokeDasharray="3 3" stroke="#f1f5f9" />
              <XAxis 
                dataKey="date" 
                tick={{ fontSize: 11, fill: '#94a3b8' }}
                axisLine={{ stroke: '#e2e8f0' }}
              />
              <YAxis 
                tick={{ fontSize: 11, fill: '#94a3b8' }}
                axisLine={{ stroke: '#e2e8f0' }}
              />
              <Tooltip content={<CustomTooltip />} />
              <Line 
                type="monotone" 
                dataKey="tarefas" 
                stroke="#6366f1" 
                strokeWidth={2.5}
                dot={{ fill: '#6366f1', strokeWidth: 2 }}
                activeDot={{ r: 6, fill: '#6366f1' }}
              />
            </LineChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>

      {/* Progresso por Responsável */}
      <Card className="shadow-sm border-slate-100 rounded-2xl lg:col-span-2">
        <CardHeader className="pb-2">
          <CardTitle className="text-lg font-semibold text-slate-800">Progresso Médio por Responsável</CardTitle>
        </CardHeader>
        <CardContent>
          <ResponsiveContainer width="100%" height={200}>
            <BarChart data={progressByResponsible} layout="vertical">
              <CartesianGrid strokeDasharray="3 3" stroke="#f1f5f9" />
              <XAxis 
                type="number" 
                domain={[0, 100]}
                tick={{ fontSize: 11, fill: '#94a3b8' }}
                axisLine={{ stroke: '#e2e8f0' }}
              />
              <YAxis 
                type="category" 
                dataKey="name" 
                tick={{ fontSize: 12, fill: '#64748b' }}
                axisLine={{ stroke: '#e2e8f0' }}
                width={80}
              />
              <Tooltip 
                content={({ active, payload }) => {
                  if (active && payload && payload.length) {
                    return (
                      <div className="bg-white p-3 rounded-xl shadow-lg border border-slate-100">
                        <p className="text-sm font-medium text-slate-800">{payload[0].payload.name}</p>
                        <p className="text-sm text-slate-600">Progresso: {payload[0].value}%</p>
                        <p className="text-sm text-slate-500">{payload[0].payload.tarefas} tarefas</p>
                      </div>
                    );
                  }
                  return null;
                }}
              />
              <Bar dataKey="progresso" fill="#6366f1" radius={[0, 8, 8, 0]} />
            </BarChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>

      {/* Distribuição por Categoria */}
      <Card className="shadow-sm border-slate-100 rounded-2xl lg:col-span-2">
        <CardHeader className="pb-2">
          <CardTitle className="text-lg font-semibold text-slate-800">Distribuição por Categoria</CardTitle>
        </CardHeader>
        <CardContent>
          <ResponsiveContainer width="100%" height={300}>
            <BarChart data={categoryData}>
              <CartesianGrid strokeDasharray="3 3" stroke="#f1f5f9" />
              <XAxis 
                dataKey="name" 
                tick={{ fontSize: 10, fill: '#94a3b8' }}
                axisLine={{ stroke: '#e2e8f0' }}
                angle={-45}
                textAnchor="end"
                height={80}
              />
              <YAxis 
                tick={{ fontSize: 11, fill: '#94a3b8' }}
                axisLine={{ stroke: '#e2e8f0' }}
              />
              <Tooltip 
                content={({ active, payload }) => {
                  if (active && payload && payload.length) {
                    return (
                      <div className="bg-white p-3 rounded-xl shadow-lg border border-slate-100 max-w-xs">
                        <p className="text-sm font-medium text-slate-800">{payload[0].payload.fullName}</p>
                        <p className="text-sm text-slate-600">{payload[0].value} tarefas</p>
                      </div>
                    );
                  }
                  return null;
                }}
              />
              <Bar dataKey="value" fill="#8b5cf6" radius={[8, 8, 0, 0]} />
            </BarChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>
    </div>
  );
}