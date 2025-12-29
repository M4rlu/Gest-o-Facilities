'use client';

import React, { useState, useEffect } from 'react';
import { 
  Menu, X, Moon, Sun, Settings, Plus, 
  Trash2, CheckSquare, Square, Edit3, Save 
} from 'lucide-react';
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

// Utilitário para classes condicionais
function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// --- TIPAGEM ---
type CheckItem = {
  id: string;
  text: string;
  checked: boolean;
};

type Topic = {
  id: string;
  title: string;
  items: CheckItem[];
};

type Module = {
  id: string;
  name: string;
  topics: Topic[];
};

// --- DADOS INICIAIS (FSSC 22000 & LIMPEZA) ---
const INITIAL_DATA: Module[] = [
  {
    id: 'mod-1',
    name: 'Cozinha (FSSC 22000)',
    topics: [
      {
        id: 'top-1',
        title: 'Higiene Pessoal',
        items: [
          { id: 'it-1', text: 'Uniformes limpos e sem botões', checked: false },
          { id: 'it-2', text: 'Ausência de adornos (joias, relógios)', checked: false },
          { id: 'it-3', text: 'Lavagem de mãos correta observada', checked: false },
        ]
      },
      {
        id: 'top-2',
        title: 'Controle de Temperatura',
        items: [
          { id: 'it-4', text: 'Buffet quente > 60°C', checked: false },
          { id: 'it-5', text: 'Refrigeradores < 5°C', checked: false },
        ]
      }
    ]
  },
  {
    id: 'mod-2',
    name: 'Limpeza Fábrica',
    topics: [
      {
        id: 'top-3',
        title: 'Áreas de Circulação',
        items: [
          { id: 'it-6', text: 'Corredores livres de obstrução', checked: false },
          { id: 'it-7', text: 'Ausência de poeira em extintores', checked: false },
        ]
      }
    ]
  }
];

export default function FacilitiesApp() {
  // --- ESTADOS ---
  const [mounted, setMounted] = useState(false); // Previne erro de hidratação
  const [modules, setModules] = useState<Module[]>(INITIAL_DATA);
  const [activeModuleId, setActiveModuleId] = useState<string>('mod-1');
  const [isSidebarOpen, setIsSidebarOpen] = useState(false);
  const [isDarkMode, setIsDarkMode] = useState(false);
  const [isEditMode, setIsEditMode] = useState(false);

  // --- PERSISTÊNCIA E MONTAGEM ---
  useEffect(() => {
    setMounted(true);
    const savedData = localStorage.getItem('facilities_data');
    const savedTheme = localStorage.getItem('facilities_theme');
    
    if (savedData) setModules(JSON.parse(savedData));
    if (savedTheme === 'dark') setIsDarkMode(true);
  }, []);

  useEffect(() => {
    if (mounted) {
      localStorage.setItem('facilities_data', JSON.stringify(modules));
      localStorage.setItem('facilities_theme', isDarkMode ? 'dark' : 'light');
      
      if (isDarkMode) document.documentElement.classList.add('dark');
      else document.documentElement.classList.remove('dark');
    }
  }, [modules, isDarkMode, mounted]);

  if (!mounted) return null; // Previne Tela Preta/Flash

  // --- FUNÇÕES DE LÓGICA ---
  const activeModule = modules.find(m => m.id === activeModuleId) || modules[0];

  const toggleCheck = (topicId: string, itemId: string) => {
    setModules(prev => prev.map(mod => {
      if (mod.id !== activeModuleId) return mod;
      return {
        ...mod,
        topics: mod.topics.map(topic => {
          if (topic.id !== topicId) return topic;
          return {
            ...topic,
            items: topic.items.map(item => 
              item.id === itemId ? { ...item, checked: !item.checked } : item
            )
          };
        })
      };
    }));
  };

  const updateText = (type: 'module' | 'topic' | 'item', id: string, newText: string, parentId?: string) => {
    setModules(prev => prev.map(mod => {
      if (type === 'module' && mod.id === id) return { ...mod, name: newText };
      
      // Se não for módulo, precisamos ir mais fundo
      if (mod.id === activeModuleId || (type === 'module')) {
        return {
          ...mod,
          topics: mod.topics.map(topic => {
            if (type === 'topic' && topic.id === id) return { ...topic, title: newText };
            
            if (type === 'item' && parentId === topic.id) {
              return {
                ...topic,
                items: topic.items.map(item => item.id === id ? { ...item, text: newText } : item)
              };
            }
            return topic;
          })
        };
      }
      return mod;
    }));
  };

  const addItem = (topicId: string) => {
    setModules(prev => prev.map(mod => {
      if (mod.id !== activeModuleId) return mod;
      return {
        ...mod,
        topics: mod.topics.map(topic => {
          if (topic.id !== topicId) return topic;
          return {
            ...topic,
            items: [...topic.items, { id: `new-${Date.now()}`, text: 'Nova inspeção...', checked: false }]
          };
        })
      };
    }));
  };

  const addTopic = () => {
    setModules(prev => prev.map(mod => {
      if (mod.id !== activeModuleId) return mod;
      return {
        ...mod,
        topics: [...mod.topics, { id: `topic-${Date.now()}`, title: 'Nova Área/Tópico', items: [] }]
      };
    }));
  };

  const addModule = () => {
    const newId = `mod-${Date.now()}`;
    setModules([...modules, { id: newId, name: 'Novo Setor', topics: [] }]);
    setActiveModuleId(newId);
  };

  // --- RENDERIZAÇÃO ---
  return (
    <div className={cn("min-h-screen flex bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-slate-100 transition-colors")}>
      
      {/* SIDEBAR - MOBILE OVERLAY */}
      {isSidebarOpen && (
        <div 
          className="fixed inset-0 bg-black/50 z-20 md:hidden"
          onClick={() => setIsSidebarOpen(false)}
        />
      )}

      {/* SIDEBAR - MAIN */}
      <aside className={cn(
        "fixed md:sticky top-0 left-0 z-30 h-screen w-72 bg-white dark:bg-slate-950 shadow-xl transform transition-transform duration-300 ease-in-out md:translate-x-0 border-r border-slate-200 dark:border-slate-800 flex flex-col",
        isSidebarOpen ? "translate-x-0" : "-translate-x-full"
      )}>
        <div className="p-6 border-b border-slate-200 dark:border-slate-800 flex justify-between items-center">
          <h1 className="text-xl font-bold bg-gradient-to-r from-blue-600 to-indigo-600 bg-clip-text text-transparent">
            Facilities Manager
          </h1>
          <button onClick={() => setIsSidebarOpen(false)} className="md:hidden">
            <X className="w-6 h-6" />
          </button>
        </div>

        <nav className="flex-1 overflow-y-auto p-4 space-y-2">
          <p className="text-xs font-semibold text-slate-400 uppercase tracking-wider mb-3 px-2">Módulos</p>
          {modules.map(mod => (
            <div key={mod.id} className="flex items-center group">
              <button
                onClick={() => {
                  setActiveModuleId(mod.id);
                  setIsSidebarOpen(false); // Fecha no mobile ao selecionar
                }}
                className={cn(
                  "flex-1 text-left px-4 py-3 rounded-xl transition-all font-medium text-sm flex items-center gap-3",
                  activeModuleId === mod.id 
                    ? "bg-blue-600 text-white shadow-lg shadow-blue-500/30" 
                    : "hover:bg-slate-100 dark:hover:bg-slate-800 text-slate-600 dark:text-slate-400"
                )}
              >
                {isEditMode ? (
                  <input 
                    className="bg-transparent border-b border-white/50 w-full outline-none"
                    value={mod.name}
                    onChange={(e) => updateText('module', mod.id, e.target.value)}
                    onClick={(e) => e.stopPropagation()}
                  />
                ) : (
                   mod.name
                )}
              </button>
            </div>
          ))}
          
          {isEditMode && (
            <button 
              onClick={addModule}
              className="w-full mt-4 flex items-center justify-center gap-2 text-sm text-blue-500 border border-dashed border-blue-500/50 p-2 rounded-xl hover:bg-blue-50 dark:hover:bg-blue-900/20"
            >
              <Plus size={16} /> Novo Módulo
            </button>
          )}
        </nav>

        <div className="p-4 border-t border-slate-200 dark:border-slate-800">
           <div className="flex items-center justify-between px-2">
              <span className="text-sm font-medium">Modo Edição</span>
              <button 
                onClick={() => setIsEditMode(!isEditMode)}
                className={cn(
                  "p-2 rounded-full transition-colors",
                  isEditMode ? "bg-amber-100 text-amber-600" : "bg-slate-100 dark:bg-slate-800 text-slate-500"
                )}
              >
                {isEditMode ? <Save size={18} /> : <Settings size={18} />}
              </button>
           </div>
        </div>
      </aside>

      {/* CONTEÚDO PRINCIPAL */}
      <main className="flex-1 flex flex-col h-screen overflow-hidden">
        {/* HEADER */}
        <header className="h-16 border-b border-slate-200 dark:border-slate-800 bg-white/80 dark:bg-slate-900/80 backdrop-blur-md flex items-center justify-between px-6 sticky top-0 z-10">
          <div className="flex items-center gap-4">
            <button onClick={() => setIsSidebarOpen(true)} className="md:hidden p-2 rounded-lg hover:bg-slate-100 dark:hover:bg-slate-800">
              <Menu className="w-6 h-6" />
            </button>
            <h2 className="text-lg font-semibold truncate hidden sm:block">
              {activeModule.name}
            </h2>
          </div>

          <div className="flex items-center gap-2">
            <button 
              onClick={() => setIsDarkMode(!isDarkMode)}
              className="p-2 rounded-full hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors text-slate-600 dark:text-slate-300"
            >
              {isDarkMode ? <Sun size={20} /> : <Moon size={20} />}
            </button>
          </div>
        </header>

        {/* ÁREA DE SCROLL */}
        <div className="flex-1 overflow-y-auto p-4 md:p-8 pb-20">
          <div className="max-w-4xl mx-auto space-y-8">
            
            {/* Título Mobile */}
            <h2 className="text-2xl font-bold md:hidden mb-6">{activeModule.name}</h2>

            {activeModule.topics.map(topic => (
              <div key={topic.id} className="bg-white dark:bg-slate-800 rounded-2xl p-6 shadow-sm border border-slate-100 dark:border-slate-700">
                <div className="flex justify-between items-center mb-4">
                  {isEditMode ? (
                     <input 
                       className="text-lg font-bold bg-transparent border-b border-slate-300 dark:border-slate-600 w-full outline-none pb-1"
                       value={topic.title}
                       onChange={(e) => updateText('topic', topic.id, e.target.value)}
                     />
                  ) : (
                    <h3 className="text-lg font-bold text-slate-800 dark:text-slate-200 flex items-center gap-2">
                      <span className="w-2 h-6 bg-blue-500 rounded-full inline-block"></span>
                      {topic.title}
                    </h3>
                  )}
                  {isEditMode && (
                    <button onClick={() => addItem(topic.id)} className="text-blue-500 hover:bg-blue-50 p-1 rounded">
                      <Plus size={20} />
                    </button>
                  )}
                </div>

                <div className="space-y-3">
                  {topic.items.map(item => (
                    <div 
                      key={item.id} 
                      className={cn(
                        "flex items-start gap-3 p-3 rounded-lg transition-all",
                        item.checked ? "bg-green-50 dark:bg-green-900/10" : "hover:bg-slate-50 dark:hover:bg-slate-700/30"
                      )}
                    >
                      <button 
                        onClick={() => toggleCheck(topic.id, item.id)}
                        className={cn(
                          "mt-1 transition-colors",
                          item.checked ? "text-green-500" : "text-slate-300 dark:text-slate-500 hover:text-blue-500"
                        )}
                      >
                        {item.checked ? <CheckSquare size={22} /> : <Square size={22} />}
                      </button>
                      
                      {isEditMode ? (
                        <input 
                          className="flex-1 bg-transparent border-b border-slate-200 outline-none text-sm py-1"
                          value={item.text}
                          onChange={(e) => updateText('item', item.id, e.target.value, topic.id)}
                        />
                      ) : (
                        <span className={cn(
                          "flex-1 text-sm leading-6 cursor-pointer select-none",
                          item.checked ? "text-slate-400 line-through" : "text-slate-700 dark:text-slate-300"
                        )} onClick={() => toggleCheck(topic.id, item.id)}>
                          {item.text}
                        </span>
                      )}
                    </div>
                  ))}
                  {topic.items.length === 0 && !isEditMode && (
                    <p className="text-sm text-slate-400 italic pl-2">Nenhum item nesta lista.</p>
                  )}
                </div>
              </div>
            ))}

            {isEditMode && (
              <button 
                onClick={addTopic}
                className="w-full py-4 border-2 border-dashed border-slate-300 dark:border-slate-700 rounded-2xl text-slate-500 hover:bg-slate-50 dark:hover:bg-slate-800 transition-colors flex flex-col items-center justify-center gap-2"
              >
                <Plus size={24} />
                <span className="font-medium">Adicionar Novo Tópico/Área</span>
              </button>
            )}
            
            {/* Espaço extra para footer */}
            <div className="h-10"></div>
          </div>
        </div>
      </main>
    </div>
  );
}
